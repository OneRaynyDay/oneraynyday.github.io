---
published: true
title: Analyzing The Simplest C++ Program - WIP
use_math: true
category: dev
layout: default
---

Here I have the simplest C++ program in existence:

```
// main.cpp
int main(){}
```

and here is the corresponding Makefile for this program (with some utilities we'll use later):

```
# Makefile
# We are running g++-9, v9.3.0
CC=g++
# Turn off optimizations because we want to be able to follow the assembly.
FLAGS=-O0 -fverbose-asm -no-pie

main: main.cpp
	$(CC) $(FLAGS) -o $@ $^ 

dump: main
	# -r shows symbol names on relocations (so you'd see puts in the call instruction below)
	# -R shows dynamic-linking relocations / symbol names (useful on shared libraries)
	# -C demangles C++ symbol names
	# -w is "wide" mode: it doesn't line-wrap the machine-code bytes
	# -Mintel: use GAS/binutils MASM-like .intel_syntax noprefix syntax instead of AT&T
	# -S: interleave source lines with disassembly.
	objdump -drwC -Mintel main &> main.dump

linker: main.cpp
	$(CC) $(FLAGS) -o /dev/null -x c $^ -Wl,--verbose

clean:
	rm main main.dump
```

When we run the program, the program simply starts up, does nothing, and returns with exit code 0. However, even this seemingly simple process is extremely complicated and as engineers we should really understand the process to protect ourselves against security vulnerabilities, to optimize our programs and to debug the annoying linker errors that plague our typical C++ development cycles. To do this, let's analyze the structure of the executable and read some assembly.

# Table of Contents

* TOC
{:toc}

# Structure: ELF files & Linkers

**ELF, which stands for Executable and Linkable Format** is the format used for binaries and libraries that we compile with C and C++. It's in an unreadable binary format that can be analyzed with several GNU tools. To understand what the assembly outputs are, we must first be familiar with the general layout of an ELF file.

You can identify an ELF file by using the `file` command:

```
❯ file main
main: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=921d352e49a0e4262aece7e72418290189520782, for GNU/Linux 3.2.0, not stripped
```

If it does say `ELF`, you can use `readelf` to analyze the headers like so:

```
❯ readelf -e main
ELF Header:
...

Section Headers:
...

Key to Flags:
...

Program Headers:
...

 Section to Segment mapping:
  Segment Sections...
...

```

If we look at the ELF Header and Program Headers sections, we can get a lot of information about the runtime behavior of the executable, so let's analyze it.

## ELF Headers

We see the following relevant details in the `ELF Header` section:

```
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
...
  Type:                              DYN (Shared object file)
  Version:                           0x1
  Entry point address:               0x1020
  Start of program headers:          64 (bytes into file)
  Start of section headers:          14584 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         11
  Size of section headers:           64 (bytes)
  Number of section headers:         27
  Section header string table index: 26
...
```

Let's go through some of these sections:

- The `Magic` field contains the letters `ELF`, denoted by hexadecimals `45 4c 46`. To identify whether this file is an ELF format, you need to check the header magic here.
- The `Class` field tells us whether this file should be run on a 32-bit or 64-bit architecture. Modern CPU's are 64-bit, which means the word size is 8 bytes rather than 4 bytes, the addressable memory space is $2^{64}$ bytes(which is practically infinite for any kind of storage) as opposed to $2^{32}$ bytes(which is 4GB, which is roughly the size of a medium dataset for machine learning), and registers can hold 64 bit data.
- The `Data` field tells us that the data is stored in `little endian`:

```
# Little endian has the most significant byte last:
Consider an 4-byte integer represented by: 0x11 0x22 0x33 0x44
The little endian representation (in increasing memory order): 0x44 0x33 0x22 0x11.
```

and `2's complement` is the representation of signed numbers. For any arbitrary positive number $N$ represented in binary, the corresponding $-N$ is represented as bit-flipped $N$ plus 1.

- The `Type` field tells us what kind of file it is. You might be surprised in this case to see that it's a `DYN` for shared object file when we're actually creating an executable. I was also baffled and asked on StackOverflow [here](https://stackoverflow.com/questions/61567439/why-is-my-simple-main-programs-elf-header-say-its-a-dyn-shared-object-file). TL;DR: We'll be adding the flag `-no-pie` to turn it into an `EXEC` type file. The rest of the blog will be based off of that assumption.

## Program Headers

We see the following relevant details in the `Program Header` section:

```
Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000400040 0x0000000000400040
                 0x0000000000000268 0x0000000000000268  R      0x8
  INTERP         0x00000000000002a8 0x00000000004002a8 0x00000000004002a8
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           ...                                    R
  LOAD           ...                                    R E
  LOAD           ...                                    R
  LOAD           ...                                    R W
  DYNAMIC        ...
  NOTE           ...
  GNU_EH_FRAME   ...
  GNU_STACK      ...
  GNU_RELRO      ...
```

Note that the rest of the table is redacted for readability. The program headers give hints to the memory loader for the process to put chunks of the program in the right places.

For a simple program, there appears to be a lot of program headers. Let's analyze what segments each of these headers point to and why they're needed.

### PHDR

`PHDR` is a bit of a strange one. According to the official linux documentations [here](http://man7.org/linux/man-pages/man5/elf.5.html), it says:

> ... specifies the location and size of the program header table itself, both in the file and in the memory image of the program.

which means it describes the actual program header table's location. According to the question I asked [here](https://stackoverflow.com/questions/61568612/is-jumping-over-removing-phdr-program-header-in-elf-file-for-executable-ok-if/61568759#61568759), it seems like the struct `PT_PHDR` is used for determining where the PIE executable was loaded in the virtual memory.

### INTERP

This specifies where the interpreter is for running shared library executables. If you look carefully, it says that the dynamic linker resides in `/lib64/ld-linux-x86-64.so.2`. Let's call it and see what it says:

```
❯ /lib64/ld-linux-x86-64.so.2
Usage: ld.so [OPTION]... EXECUTABLE-FILE [ARGS-FOR-PROGRAM...]
You have invoked `ld.so', the helper program for shared library executables... This helper program loads the shared libraries needed by the program executable, prepares the program
to run, and runs it.  You may invoke this helper program directly from the command line to load and run an ELF executable file; this is like executing that file itself, but always uses this helper program from the file you specified, instead of the helper program file specified in the executable file you run.
```

TL;DR: Programs that load shared libraries will invoke this dynamic linker to run the shared library executable. You usually don't call this yourself, but you can.

### LOAD

Load basically tells the linker to allocate a particular segment of memory with particular permissions. In the above, we see that there are 4 `LOAD` sections. This only happens for the newer versions of `ld`. These segments in C++ are for the following:

- `.text`, which holds the code to be executed. This should be in the `R E` section.
- `.rodata`, which means *read-only data*. This usually holds static constants that are used in the program. This is in one of the `R` sections.
- `.data`, which is read/write data containing the heap and the stack (usually). This is in the `RW` section. There's no execute because of buffer overflow security vulnerabilities leading to execution of code in the data section. In addition, we have `.bss` in this section as well. The name doesn't really mean too much now - you should just consider it as "zero-initialized data". It contains global variables and static variables that are zero-initialized. *The reason this segment exists is for space optimization in the executable itself*. (Imagine a lot of zero buffers adding space to the executable's size)
- The ELF header information is in the other `R` section.

### DYNAMIC

If this executable requires further dynamic linking, this field will point to us exactly what information is required. Usually, you don't look at this, but rather the `.dynamic` section in the ELF file to see what shared libraries are required. In reality, you don't need to do this - instead, use `ldd` to find the dependencies yourself:

```
❯ ldd main
	linux-vdso.so.1 (0x00007fff23f7a000)
	libstdc++.so.6 => /usr/lib/libstdc++.so.6 (0x00007ff8ae1cc000)
	libm.so.6 => /usr/lib/libm.so.6 (0x00007ff8ae086000)
	libgcc_s.so.1 => /usr/lib/libgcc_s.so.1 (0x00007ff8ae06c000)
	libc.so.6 => /usr/lib/libc.so.6 (0x00007ff8adea6000)
	/lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007ff8ae3d9000)

```

We see that even though we're not really running anything in our program, we're still pulling in these `.so`'s. These core libraries are required for the C++ runtime. Linker issues are the biggest headache, involving `rpath, runpath, LD_LIBRARY_PATH`, and other variables that may or may not be baked into the `.dynamic` section of the ELF file. I highly recommend this [blogpost](https://amir.rachum.com/blog/2016/09/17/shared-libraries/) if you're running into a practical issue with dynamic linking `.so` files.

### NOTE 

This is a very free-style section used by vendors or engineers to mark an object file with special information. This information is usually used to check for compatibility. A note segment is fairly small and we don't really need to care much about this.

### GNU_EH_FRAME

In here, `EH` stands for exception handling. This is a sorted data structure that handles exceptions. It maps particular exceptions to particular function handlers, and helps with frame unwinding for those nice backtraces you get with `bt` in `gdb`.

### GNU_STACK

This is a lightweight header that tells us what permissions we gave the stack. The stack in general *should not be executable* for security vulnerabilities, so let's see whether our binary is safe with `scanelf`:

```
❯ scanelf . -e
 TYPE   STK/REL/PTL FILE 
ET_EXEC RW- R-- RW- ./main
```

We don't allow execution on the stack - good news!

### GNU_RELRO

This particular section is purely for protection against security vulnerabilities. ELF binaries have two maps called the **Global Offset Table**, otherwise known as GOT, and the **Procedure Linkage Table**, otherwise known as PLT. The GOT stores the exact address of global variables, functions and other symbols from a dynamically loaded shared library. These values in the GOT are subject to change when you re-compile the same program. When calling a function from the shared library, there may be many functions that are never called. To reduce the function resolution overhead, we create stubs in GOT for *lazy loaded functions*. The steps to resolve a lazy loaded function in runtime is outlined below:

- User calls a lazy loaded function from a dynamically loaded library. This is an entry in the GOT (fixed, unlike the address of the actual function).
- If the function is unresolved, stub code in the section jump to the PLT, where it redirects the call to the dynamic linker's `_dl_runtime_resolve` routine.
- `_dl_runtime_resolve` populates the GOT section with the correct address of the function.
- The address of the function is used.
- Upon further invocation of the function, the GOT no longer needs to jump to the PLT, and immediately gives the address of the function.

The reason I explained the above is to illustrate how important the PLT and GOT are, but I still haven't really explained what `GNU_RELRO` is. `RELRO` in this case stands for **RELocation Read Only**. These in-memory structures are read-write in order to save the resolved addresses of the loaded functions, but that lends itself to security vulnerabilities. What if the user can buffer-overflow and change the entry in the GOT to execute a function containing arbitrary code?

Well, one way is to make the function information loading _all eager_, and then turn the section read-only before the user can screw around with the GOT. Before user code can be executed, if the GOT is already populated then turning this section read-only with the [system call](http://man7.org/linux/man-pages/man2/mprotect.2.html) `mprotect` will prevent any vulnerabilities.

## Linker Script

So in the above section `DYNAMIC` we talked a bit about linkers. We also see that there are several sections in our code as well as dynamically loading from `libc.so`, `libstdc++`, etc. Where are the dynamically loaded libraries' data going to be placed in the final layout of our executable? If we use the below flags with verbose linkage, we'll see the **linker script** actually being emitted (major parts redacted):

```
❯ make linker
g++ -O0 -fverbose-asm -no-pie -o /dev/null -x c main.cpp -Wl,--verbose
...
using internal linker script:
==================================================
...
OUTPUT_FORMAT("elf64-x86-64", "elf64-x86-64", "elf64-x86-64")
OUTPUT_ARCH(i386:x86-64)
ENTRY(_start)
SEARCH_DIR("/usr/x86_64-pc-linux-gnu/lib64"); ...
SECTIONS
{
  ...
  .interp         : { *(.interp) }
  ...
  .rela.dyn       :
    {
      *(.rela.init)
      *(.rela.text .rela.text.* .rela.gnu.linkonce.t.*)
      ...
      *(.rela.lrodata .rela.lrodata.* .rela.gnu.linkonce.lr.*)
      *(.rela.ifunc)
    }
  ...
  .plt            : { *(.plt) *(.iplt) }
  .plt.got        : { *(.plt.got) }
  .plt.sec        : { *(.plt.sec) }
  ...
  .init_array    :
  {
    PROVIDE_HIDDEN (__init_array_start = .);
    KEEP (*(SORT_BY_INIT_PRIORITY(.init_array.*) SORT_BY_INIT_PRIORITY(.ctors.*)))
    KEEP (*(.init_array EXCLUDE_FILE (*crtbegin.o *crtbegin?.o *crtend.o *crtend?.o ) .ctors))
    PROVIDE_HIDDEN (__init_array_end = .);
  }
  .fini_array    :
  {
    ...
  }
  ...
}
```

As expected, the linker script tells us at the beginning that we are generating a 64-bit ELF file made for the 64 bit architecture. Upon execution of the program, we start at a symbol called `_start`, and our shared objects are found in the `SEARCH_DIR` paths(which can be modified by the `rpath` or `runpath` variables during compilation). Then, the linker script describes exactly how the sections are laid out in our executable.

Understanding the syntax for this linker script is actually not too hard. When we look at the `INTERP` section:

```
  .interp         : { *(.interp) }
```

This is saying the `.interp` section is laid out by all the `.interp` sections the linker was able to find (in the other shared libraries) in some sequence. The `*` matches all section definitions found and the `(.interp)` selects that particular section specification. Similarly, we see the `.rela.dyn` section being defined by all of the `.rela.init` sub-sections first, and then the `.rela.text` sections laid out after. The reason the linker did not say `.rela.dyn : { *(.rela) }` is because it would've been laid out without the `init` subsections being grouped together.

Then, we see the PLT sections along with the GOT being laid out with `.plt`, `.plt.got` and `.plt.sec` sections defined.

**Then, we see two sections that we weren't familiar with before - `.init_array` and `.fini_array`.** What are these? TODO