# FreeNOS Memory Management by Vaibhavi Jhawar 

Introduction

Process memory management is essential to the existence of an operating system; it ensures that all processes have the memory resources they need in order to execute. It also prevents processes from accessing or modifying any memory that may belong to other processes, saving the operating system from breaking down due to vulnerabilities in memory allocation, protection, and performance. 

While taking a hands-on approach to learning operating systems, FreeNOS, an open-source operating system designed for educational purposes, is popularly used. In this report, we will explore different process memory management techniques that an operating system utilizes by walking through code written for memory management in the FreeNOS operating system. 

FreeNOS Memory Management System

There are several types of memory management systems: single contiguous allocation partitioned allocation, segmented memory management, and paged memory management. Single contiguous allocation is when there is a small portion of computer memory that is reserved for the operating system, but the rest is available for a single application. The MS-DOS, an operating system released in 1981, uses this memory allocation system. Partitioned allocation divides physical memory into multiple fixed-size partitions that can be allocated to a process. The IBM System/360 most popularly used this system. Segmented memory allocation divides memory into segments of variable sizes, each segment is then allocated to a specific process of a program. Early Microsoft Windows operating systems used segmented memory management systems. Lastly, the paged memory management divides the computer’s memory into “page frames”, fixed-size units of memory, and separates virtual address spaces into pages of the same size. FreeNOS, like most modern operating systems,  uses a paging-based virtual memory management system for process memory management. 

Process memory management is the way a computer operating system manages the memory used by programs and application executions. The management system is in charge of freeing up memory when it is no longer being used or requested. In FreeNOS, the paged memory management allocates memory to a process using a virtual memory system, creating the illusion that each process has its own private address space. The SplitAllocator class is in charge of separating kernel-mapped memory at virtual and physical addresses. It takes in a physical memory range, a virtual memory range, and the size of a memory page. The class also contains methods for allocating deallocating physical and virtual memory, releasing memory pages, and converting between physical and virtual addresses. It also uses the BitAllocator class, which is responsible for managing physical memory allocation. 

<img width="497" alt="Screenshot 2023-05-05 at 9 31 16 PM" src="https://user-images.githubusercontent.com/31997056/236600125-d300ffb3-d7fb-4a92-844a-68089745692f.png">

The BitAllocator class allocated memory using a BitArray where all memory is divided in equal-sized chunks. 1 in a BitArray means the chunk is being used and 0 means it's unused. This class is a subclass for the Allocator class, an interface that is used by the main PoolAllocator class. Its constructor takes in three arguments, a range that specifies the memory it should manage, the chunkSize that specifies how much memory is allocated to each chunk, and a pointer to the existing bitmap array, or ZERO to allocate a new bitmap array. 

<img width="496" alt="Screenshot 2023-05-05 at 9 31 28 PM" src="https://user-images.githubusercontent.com/31997056/236600138-91524d4e-1388-431f-b6af-d2f6e197bb80.png">


FreeNOS uses an implementation of a memory allocator that uses pools to manage same-sized objects. In the PoolAllocator.h class, pools are allocated memory at the size of a power of 2, and each pool is pre-allocated and has a bitmap that represents free blocks. The minimum size for a pool is 2^2 bits, and the maximum size is 2^27 bits or 128 MiB. 

<img width="497" alt="Screenshot 2023-05-05 at 9 31 36 PM" src="https://user-images.githubusercontent.com/31997056/236600147-a95c4416-72ff-4830-88f2-6834fbeebae0.png">


The Pool struct is used to represent each pool of same-sized objects. This class includes several helper functions that are used to calculate object size, minimum object count, and total memory usage. The PoolAllocator class inherits from the Allocator class, an abstract class that defines the interface for memory allocation and deallocation. In the allocator class, memory allocators form a hierarchy of parent-child. A parent allocator is able to allocate memory to a child allocator, and if a child allocator runs out, it can ask the parent for more memory. One parent allocator typically manages multiple memory pools, whereas child allocators are used to subdivide the memory pool being managed by the parent allocator to apply different policies, and management strategies for different parts of a program and subsets of memory. 

<img width="493" alt="Screenshot 2023-05-05 at 9 31 46 PM" src="https://user-images.githubusercontent.com/31997056/236600149-d307b488-16b6-44a3-bd0b-87c1a891bd57.png">


Lastly, we have a BubbleAllocator class, which manages a block of contiguous memory and keeps allocating memory at the end of this block. It does not free memory and is essential if providing fast allocation memory chunks varying in size. The allocator keeps track of the last allocated memory address and when a new memory request is made, the allocator returns the next available chunk of memory. It provides an alternative to heap memory allocation and is efficient and lightweight for dynamic memory allocation. The BubbleAllocator does not provide the ability to free individual memory. 

<img width="497" alt="Screenshot 2023-05-05 at 9 39 45 PM" src="https://user-images.githubusercontent.com/31997056/236600185-4607e612-6a4b-4fef-91ee-96fec7b25629.png">


The process memory management system in FreeNOS consists of five classes, Allocator, BitAllocator, BubbleAllocator, PoolAllocator, and SplitAllocator. All five are essential to the efficiency of allocating and deallocating memory resources to and from processes. 


Discussion and Analysis

Advantages and Disadvantages of Memory Management Systems

Bit Allocator: The BitAllocator uses memory efficiently by allocating only the required number of chunks and avoiding fragmentation. It is especially useful when the size of chunks is small and the number of chunks is large. It is also fast and simple and easy to implement. However, its fixed chunk size cannot be dynamically changed, and if the application needs to allocate a chunk of a different size, it would need to create a separate BitAllocator for that size, wasting effort and memory. Also, if the size of the chunks is small, the memory used to store the bitmaps can be significant and can result in wasted memory. It can also create fragmentation within chunks if chunks are partially used. 

Bubble Allocator: In most cases, the BubbleAllocator is very efficient in terms of memory usage because it allocates the first available chunk of memory that fits the requested size, minimizing fragmentation. However, it can also create many small chunks of free memory that can’t be used for larger allocations, leading to significant wasted space. The allocator is also easy to implement as it doesn’t require any complex data structures, and has fast allocation and deallocation because it only requires a simple scan through the memory block to find available memory. However, it doesn't scale well because it requires a full scan of the memory block every time an allocation is made, which is really slow when the application grows. The allocator also has limited functionality, it doesn’t support advanced features like thread safety, garbage collection, or memory pooling, which are important for some applications. 

Pool Allocator: The PoolAllocator reduces memory fragmentation by dividing memory into fix-sized blocks to ensure there are no small chunks of memory that can be used for allocation. It also provides better performance of allocation and deallocation because the allocator only needs to maintain a list of free blocks instead of searching for free memory. Also, memory usage is predictable since the allocator pre-allocates a fixed amount of memory, so it's easier to estimate how much memory will be needed for a program. However, due to fixed block size, the allocator provides less flexibility when objects of varying sizes have to be accounted for. Also, there can be some overhead in maintaining the list of free blocks and tracking which blocks are in use, which is more significant if the program allocates many small objects.

Split Allocator: A SplitAllocator reduces external fragmentation by effectively using its available memory by splitting larger memory blocks into smaller ones. It also efficiently uses memory y allocated only the memory required for each request, and makes memory allocation faster by maintaining a list of free blocks. It does however increase internal fragmentation because it may allocate more memory than necessary to satisfy a request, which occurs when the size of a requested block of memory is less than the smallest block the allocator can allocate. There is also reduced scalability because it requires exclusive access to the memory pool during allocation and deallocation time. Lastly, it is complex it implement since it requires careful management of free and allocated memory blocks to minimize fragmentation and avoid memory leaks.

How it compares to other memory management systems

One commonly used memory management system is the buddy allocator, which is often used in operating systems. The buddy allocator is a type of binary tree that recursively splits blocks of memory into smaller blocks until it reaches a block that is the correct size for the memory request. When a block is freed, the buddy allocator coalesces adjacent blocks to form larger blocks. The buddy allocator is known for being efficient and scalable, although it can suffer from internal fragmentation.

Another commonly used memory management system is the slab allocator, which is often used in Linux systems. The slab allocator divides memory into fixed-size "slabs," each of which can hold a specific type of data structure. When a new data structure is needed, the slab allocator allocates memory from the appropriate slab. The slab allocator is known for being fast and memory-efficient, although it can be less flexible than other memory management systems.

Other memory management systems include the arena allocator, the stack allocator, and the linear allocator. Each of these systems has its own strengths and weaknesses, and the choice of which to use will depend on the specific needs of a given project.

The FreeNOS memory management system is predictable and deterministic, it provides predictable memory allocation and deallocation time. It is also efficient because it is designed to minimize memory fragmentation and improve memory utilization by using the PoolAllocater. It is also customizable, allowing developers to tailor the memory management to the specific needs of the application. However, it has limited support for dynamic memory allocation, limiting the flexibility of being used for the applications that require dynamic allocations. Also, it is complex to use because developers need a mandatory deep understanding of the system’s architecture and algorithms. Lastly, it has limited portability because it is designed specifically for the FreeNOS operating system. FreeNOS's memory management system is well-suited for real-time embedded systems that require predictable memory allocation and deallocation times. However, it may not be suitable for all applications, particularly those that require dynamic memory allocation or need to be portable to other operating systems. 

Conclusion

Understanding the FreeNOS operating system is challenging, but incredibly rewarding. It brings understanding and appreciation for the operating systems that our computers use today. Although the FreeNOS memory management system is both fast and efficient, I would not be able to use it in my day-to-day life. The system is not dynamic and is built for very predictable use cases. Also, it is not very portable and is difficult to use for everyday operations. It has been a pleasure learning more about FreeNOS, but I will be happy if I do not have to use FreeNOS again.  



References:

ChatGPT was used for help writing and formatting this assignment with the permission and verbal encouragement of Professor Dong

https://github.com/nieklinnenbank/FreeNOS/

https://www.geeksforgeeks.org/memory-management-in-operating-system/ 

https://en.wikipedia.org/wiki/Memory_management_(operating_systems) 
