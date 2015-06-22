#Glossary


##File Systems

* XFS - 

* ext4 -

##Memory Management

* paging - A way of organizing data into fixed-size chunks, called pages.

* page frames - The sections that compose the physical ram. One page frame holds one page of data.

* VIRT(VSIZE) - The total amount of virtual memory a process has asked for.

* RES(RSS) - The total amount of virtual memory that a process is currently mapping to physical memory. 

* Translation Lookaside buffer(TLB) - A dedicated hardware cache for storing frequently used pages.

* Minor page fault - A page is not mapped to physical address because it is newly allocated page. 

* Major page fault - A page is not mapped to physical address because it resides in swap or needs to be loaded from a file on disk. 

* page cache - Data that is read from or written to disk being stored in memory as a cache.

* NUMA - Non-uniform Memory Access system. System memory is divided into zones tht are connected directly to particular CPus or sockets.

##States of a memory page

* Free: The page is available for immediate allocation.

* Inactive clean: The page is not in active use and the content corresponds to the content on disk

* Inactive dirty: The page is not in active use but the page content has been modified since being read from disk and has not yet been written back.

* Active: The page is in active use and not a candidate for being freed.


##CPU Tuning

* Interrupt - a signal originating from hardware (hard-interrupt) or software (soft-interrupt) that there is work to be done right now.

##CPU Schduling policies

* SCHED_NORMAL or SCHED_OTHER - The default policy. Processes inside this policy will be scheduled by the Completely fair scheduler(CFS). The priority for this policy can only ever be 0, although nice(1) values are honored. Selected with chrt -o. When using systemd, this policy is the default, but it can be selected explicitly by including the line [CPUSchedulingPolicy=other](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#drop-in-tunables) inside the [Service] block

* SCHED_BATCH - The same as SCHED_OTHER, but with a higher interval in between scheduler operations. Processes in SCHED_BATCH Will always be treated as non-interactive, CPU-intensive by the scheduler. Selected with chrt -b

* SCHED_IDLE - Processes in SCHED_IDLE are treated by the scheduler as being even below nice-level 19. The static priority can only ever be 0. Selected with chrt -i, or [CPUSchedulingPolicy=idle](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#drop-in-tunables).

* SCHED_FIFO - A real-time scheduler. Priorities can range from 1 to 99, with 1 being the lowest and 99 being the highest. A process in SCHED_FIFO will stay running on a CPU until it is pre-empted by a higher priority process, blocked by an I/O request, or the process voluntarily calls sched_yield() to give up the CPU. Selected with chrt -f priority.

* SCHED_RR - An extension on SCHED_FIFO. All the same rules apply, but processes now also get a maximum time slice on the CPU before they are pre-empted. Selected with chrt -r
