The logical thought process of programmers when they try to optimize a program should be something along the lines of:

1. Measure and profile. Without knowing which part of the code takes the longest to run, there is no way to guarantee that any optimization effort will be statistically significant to the overall performance.
2. Create a benchmark, whether it be micro or not. After you've found a routine in the program that is dominating the runtime, it makes sense to create a benchmark that _tries_ to mimic the use pattern of the actual routine.
3. Hypothesize and test. After a particular optimization for the routine has a noticeable effect on the benchmark speed and is tested to be functionally equivalent, attempt to replace the routine with the optimized implementation and run the same program again. Repeat step 3 until successful.
4. Profit? If it did make a noticeable difference, you can call it a win. However, if the performance still fails to meet requirements, go back to step 1.

This may be surprising, but it should be duly noted that **measuring is the first step(and often the hardest step) for optimization**.

*This is an anecdote. YMMV*

I was trying to optimize some multi-threaded C++ routines a while ago and my go-to solution for step 1 was to use perftools (lately I've been exporting the results to [flame graphs](http://www.brendangregg.com/flamegraphs.html)) which showed me which subroutines were taking the most time. There were a few places where I enabled [SIMD optimizations](https://en.wikipedia.org/wiki/SIMD) by making element-wise operations independent in loops, and cached high-level intermediate results. Things were going swell, until what was taking up >30% of my entire runtime was load/store instructions(L1/L2/L3 cache misses and page faults). I stopped there because I didn't have a clear picture of the memory layout and whether there was false-sharing, lacking data locality, or memory fragmentation in my program.

Erata is a tool that allows you to visualize your memory layout. By giving the developers a visualization of allocations and deallocations(whether on the heap or not) through time, it gives them the ability to **measure and profile** the program's memory accesses. There are existing tools that help with analyzing memory usage, but we'll outline what Erata has that makes it a modern solution.

## Massif

[Massif](https://valgrind.org/docs/manual/ms-manual.html) is a heap profiler that gives useful diagnostic information about the size of heap allocations over time. To use it on a binary `program`, simply run `valgrind --tool=massif ./prog` and you'll generate some heap utilization data. What's awesome about it is the amount of community tooling it has. For example, `massif-visualizer` is a beautiful visualization of the "memory vs. time" graph. Here's an example:

![massif-visualizer]({{ site.url }}/assets/massif-visualizer.png)

# <script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
