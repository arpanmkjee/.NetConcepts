# How does the Garbage Collector work ?

Every program needs memory. Unfortunately, memory is finite.

Software must cope with memory usage, and there are two ways to manage it: manually and automatically. Manual memory management is prone to errors, especially with exceptions and asynchronous code. This is why modern managed environments (.NET, Erlang, and many more) implement automatic memory management with garbage collection.

enter image description hereFigure 1

enter image description hereFigure 2

No object or application root in Figure 2 has a reference to Object D or Object H. In the garbage collection process, the garbage collector discards both of these objects then compacts the heap.

There is a potential issue with this scenario. Assuming Object E occupies a large amount of memory, e.g. 300 MB, the performance will be suboptimal when the garbage collector moves Object E to **Object D’**s memory location.

## In .NET objects are divided into two groups:

Large Objects – larger than 85KB or multidimensional arrays.
Small Objects – everything else.

## Large Objects

The Common Language Runtime (CLR) allocates large objects on the Large Object Heap (LOH). The primary difference with the previous algorithm is that the garbage collector never compacts this heap. This increases performance, but there is a catch. There may not be enough room for a new object even with enough available memory. This causes the CLR to throw an inappropriately named OutOfMemoryException.

enter image description here Figure 3

## Small Objects

The garbage collector uses three segments, called generations, for small objects:

    Generation 0
    Generation 1
    Generation 2

The CLR allocates memory for new objects in Generation 0. When this heap is full, the garbage collector discards objects no longer in use and promotes surviving objects (referred to as live objects) to Generation 1. The Generation 0 heap is then available to hold more newly created objects.

When insufficient, the garbage collector repeats this process for Generation 1, and if still insufficient, continues with Generation 2. The garbage collector removes non-referenced objects and promotes surviving objects for Generation 1. This behavior differs in Generation 2 as it contains the longest surviving objects. After garbage collection completes, the garbage collector compacts the heaps.

enter image description here Figure 4

In Figure 4, an application root references Object F in Generation 0 and the garbage collector promotes it to Generation 1. Before the next garbage collection, the application root dereferences Object F, and the garbage collector frees it but promotes objects reachable from an application root to Generation 2.
