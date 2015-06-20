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

##States of a memory page

* Free: The page is available for immediate allocation.

* Inactive clean: The page is not in active use and the content corresponds to the content on disk

* Inactive dirty: The page is not in active use but the page content has been modified since being read from disk and has not yet been written back.

* Active: The page is in active use and not a candidate for being freed.

