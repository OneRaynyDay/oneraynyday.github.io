# My Arch Linux Setup

I've set up my Arch Linux environment for over a year now and I haven't made significant changes to it due to the fact that all of my work is through my company's Macbooks (with SSH into remote compute resources, of course). I figured since the next few C++ blogs I'll be writing are going to be done with `gcc` instead of `clang`, and I will be dissecting the C++ into assembly using `objdump` instead of `otool`, I'll be using my workstation more often.



This blog is meant to illustrate how I set up my Arch Linux environment **after** I've set up the appropriate UEFI, swap, and filesystem partitions. I'm not going to explain how boot loaders work, how to set up Pacman (the package manager for Arch), setting up locale, etc. With that said, let's dive in!



# `i3`

What is `i3`? It's a **tiling window manager** that allows you to place windows efficiently on your screen, allow simple navigation through keystrokes, separating windows into workspaces, and much more. Below is a screenshot of my setup:

![screenshot1]({{ site.url }}/assets/arch_screen.png)

As you can see, I have multiple terminal windows open, as well as Spotify, neatly tiled into bifurcated windows (as I'll explain later, you can adjust the size of the windows beyond the fixed $\frac{1}{2^n}$ sizes as well). In addition, you can see the status bar above, showing the metrics on the top right and workspace on the left. We'll explain how to set this up.

## Installing `i3`

So there are two versions of `i3` as far as I know. The windows manager I'm using has gaps in between the processes and is called `i3-gaps`, and the default one is called `i3-gaps`. Install either of these with `pacman -S <i3 package>`. Upon installing, you should be able to get 



