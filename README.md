# RHCA-Performance-Study-Guide



# Chapter 2 - Collecting, Graphing, and Interpreting Data

## Units and Unit Conversions

## Profiling Tools

###VMstat
The vmstat command, if given no arguments, will print out the averages of various system statistics since boot. The vmstat command accepts two arguments. The first is the delay, and the second is the count. The delay is a value in seconds between output. The count is the number of iterations of statitics to report. If no count is given, vmstat will continously report statistics.

###Sar
The [sar](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#sar) command is a multipurpose analysis tool that is part of the systat package. It works in two modes. It can read data collected by a cron job every 10 mintues, or it can be used to collect instantaneous data about the state of the system.

The cron job is install as ```/etc/cron.d/sysstat```, which runs the /usr/lib64/sa/sa1 and /usr/lib64/sa/sa2 commands that collect data using /usr/lib64/sa/sadc and sar. This data is stored in /var/log/sa/sadd, where dd is the two-digit day of the month.

For best results when using sar, make sure to set a locale with a LANG environment variable that provides 24-hour time support.

###IOstat

###MPstat
The [mpstat](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#mpstat) command reports CPU-related stats. Like sar, it may be necessary to configure the LANG for 24-hour time.

##Plotting Data
[gnuplot](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#gnuplot) is a data plotting program that can take data which you have collected and graph it in a number of ways.

###gnuplot demonstration
1. Capture two samples of the one-, five-, and 15-minute load averages from uptime, using awk to process the output into a simpler form to handle. In the following demonstration, we will also start some arbitrary system activity with dd to put load on the system
```shell
uptime | awk '{print $1, $(NF-2), $(NF-1), $NF }' > /tmp/uptime
```
2. gnuplot uses ```set``` to assign values and plot to list which fields to plot. Next, create a text file that includes the set and plot commands. Create a file name /tmp/uptime.gnuplot that contains the following:
```shell
set xdata time
set timefmt '%H:%M:%S'
set xlabel 'Time'

set ylabel '15-minute load average'
plot '/tmp/uptime' using 1:4
```

The first three lines set the X-axis to a time value, set the time format, and set a label. The next line sets the Y-axis label. The final line generates a plot using the /tmp/uptime data. The X-axis will use column 1 and the Y-axis will use column 4.

3. After making sure that gnuplot package is installed, plot the data with gnuplot:

```shell
yum install -y gnuplot
gnuplot /tmp/uptime.gnuplot
```

4. Unfortunately, once the graph has been generated, gnuplot exits and so does the graph. When using gnuplot to display to the screen rather than an output file, use the -persist option to leave the graph on the screen.
```shell
gnuplot -persist /tmp/uptime.gnuplot
```

5. Set the X-axis and Y-axis to values that wil better show the data. Edit /tmp/uptime.gnuplot to contain the following:
```shell
set xdata time
set timefmt '%H:%M:%S'
set xlabel 'Time'
set format x '%H:%M:%S'

set ylabel '15-minute load average'
set yrange [0:]

plot '/tmp/uptime' using 1:4
```

##Performance Co-Pilot
Performance Co-Pilot, or pcp for short, allows admins to collect and graph data from various subsystems.

If graph data is also desired, an administrator will also need to install the pcp-gui package. pcp-gui provides additional utilities like pmchart to generate graphical data of machine metrics.

###Using the pcp command line utilities.


* [pmstat](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#pcpperformance-co-pilot) provides information similar to vmstat. Like vmstat, pmstat will support options to adjust the interval between collections (-t) or the number of samples (-s).

* Another convenient utility is [pmcollectl](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#pcpperformance-co-pilot). pmcollectl is a Python port of a popular open source storage statistics application called collectl.

* [pmatop](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#pcpperformance-co-pilot) provides a top-like output of machine statistics and data. It includes disk I/O statistics, network I/O statistics, as well as the CPU, memory, and process information provided by other tools.

Performance Co-Pilot also has a text-based query mechanism for interrogating individually tracked metrics. To obtian a list of the metrics stored in the Performance Co-Pilot database, use [pminfo](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#pcpperformance-co-pilot). [pmval](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#pcpperformance-co-pilot) can then be used with the metric name to gather data about the item.

###Using the pmchart graphing utility
Performance Co-Pilot also provides a program called ```pmchart``` that takes the gathered data and graphically represents it. ```pmchart``` is provided by the pcp-gui package.

#Chapter 3 - General Tuning

##Configuring System Tunalbes

###Configuring Kernel Tunables
The linux kernel exposes tunable parameters in /proc. It also holds information about the system.
Some examples:
cat /proc/cpuinfo will get you info about the cpu
cat /proc/meminfo will get you info about the memory.
cat /proc/swaps
cat /proc/pids/<pid#> - Systemd journal
Exsercise:
Run VIM inside screen and inspect the process systemd journal.
/proc/sys - this is where the kernel tunables are available.
Any changes in here take effect immidietly, but are not persistant.

Important subdirectories in /proc/sys:

* /etc/proc/sys/dev - Contains tunables for devices such as RAID devices, CD-ROMs, SCSI devies, and one or more prallel ports

* /etc/proc/sys/fs - Holds file-system related tunables

* /etc/proc/sys/kernel - COntains tunables that change how the kernel works internally

* /etc/proc/sys/net - Contains tunables that change network-related settings

* /etc/proc/sys/vm - Contains tunables which change the virtual memory management of the kernel.

*Always use echo to add to these files*
For example:
echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_all
cat /proc/sys/net/ipv4/tcp_[rw]mem -- these are the start,min,max.
Could change that using this command:
echo "8192 87380 6291456" > /proc/sys/net/ipv4/tcp_rmem


###Manage /proc/sys tunables with sysctl
To list all avilable tunables:
```shell
sysctl -a
```
####Change sysctl tunables by command line
```shell
sysctl -w vm.swappiness=10
```

####Change sysctl tunables by configuration file.
To permanently change /proc/sys kernel tunables, the settings need to be stored in the /etc/sysctl.conf
```shell
echo "vm.swappiness=10" >> /etc/sysctl.d/filename.conf
sysctl -p /etc/sysctl.conf - makes the change without rebooting the system
```

###Configuring module parameters
These kernel modules allow the kernel to automatically load into memory only the components that are needed for a particular system.

Many kernel modules have settings that can be changed. Some settings can only be assinged when a kernel module is loaded, while others can be set upon loading or changed on the fly.

Another way to get information about a kernel module is the [modinfo](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#modinfo) command. The ````modinfo module``` command returns the module information of a particular kernel module on the command line.

With the command ```modinfo -p module``` only the parameters of the module are shown.

The ```/sys``` directory contains information provided by the kernel about the state of the system. Look in the file ```/sys/module/modulename/parameters/parametername``` to see the current setting for a module parameter.

To permanently set the same value every time that the module is loaded, create a special configuration file in ```/etc/modprobe.d```. The new file name must end in ```.conf```

```shell
options modulename parameter=value
```

Parameters can be specified on the [modprobe](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#modprobe) command line at load time if the setting needs to be applied only once, or the parameters settings in a configuration file need to be overridden.

##Installing and Enabling tuned
```shell
systemctl enable tuned
systemctl start tuned
```

###Selecting a ```tuned``` profile
Different workloads and conflicting tuning goals for server systems, such as performance, latency, and power saving require different tuning.

The tuned package contains the [tuned-amd](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#tuned) command, which acts as an interface to change settings of the tuned daemon.

For static tuning, mostly predefined sysctl and /sys settings are appliedt to the system.
Profiles can be switched based on the current system time.
Dynamic tuning.
Some tuned profiles activate helper programs that montior the activity of selected subsystems.
/etc/tuned/tuned-main.conf is the main configuration file.

To configure dynamic tuning, change the dynamic_tuning setting to 1 in /etc/tuned/tuned_main.conf

##Creating Custom tuned Profiles
The ```tuned``` service supports the creation and use of custom tuning profiles. The tuned package ships with build-in profiles residing in ```/usr/lib/tuned/profilename/```.

The /etc/tuned directory holds the active_profile file, which contains the name of the currently active tuned profile and the tuned-main.conf configuration file for configuring dynamic tuning.

The build-in profiles should not be altered. Instead, a new directory /etc/tuned/profilename needs to be created. The name of the newly created direccotry in /etc/tuned is recognized by tuned as the profile name of the custom profile. To create a new profile called myprofile, the directory /etc/tuned/myprofile needs to be created.
```shell
mkdir /etc/tuned/myprofile
```

In the newly created directory for the custom profile, tuned expects the tuned.conf configuration file.

###Inherit tuning settings from existing profile
```shell
[main]
include=<profile-name>
```

###Specifiy Additional sections in the custom profile
```shell
[NAME]
type=<pluginname>
devices=<devices>
```
Example:
```shell
[all_sd_disks]
type=disk
devices=sd*
readahead=4096
```

###Execute a shell script with the cusotom profiles
To add the script /etc/tuned/myprofile/script.sh to the myprofile tuning profile, add the following section to /etc/tuned/myprofile/tuned.conf
```shell
[script]
script=script.sh
```

###Set sysctl parameters in the custom profile
```shell
[sysctl]
net.ipv4.icmp_echo_ignore_all=1
```

###Debugging custom tuned profiles
/var/log/tuned

##Using tuna to create a tuned profile
The GUI can show you what effect the tuning profile had.
Use this gui to active settings in a custom tuned profile.
```shell
yum install tuna
tuna -g
```

#Chapter 4 - Limiting Resource Usage

##Using ulimit
Ulimit is a mechanism to limit system resources.
Can set hard or soft limits. Soft limits can be temporaily exceeded by users.
```shell
ulimit -a
```
Configuration files:
```shell
/etc/security/limits.d/20-nproc.conf
/etc/limits.conf
```
##Configuring persistent ulimit rules
Using the pam_limits module, ulimits can be applied on a per user/group bases in:
```shell
/etc/security/limits.d/<name>.conf
```
For example, to limit all the users in the managers group to a maximum of three simultaneous logins, ```/etc/security/limits.d/managers.conf``` might include the line:
```shell
@managers hard maxlogins 3
```

###Setting limits for services
With systemd, it is also possible to set POSIX limits for services. This is accomplished using the family of Limit*= entries in the [Service] block of a unit file.

Unit Files:
```shell
/usr/lib/systemd/system*
/etc/systemd/system/*
man systemd.service
```
DON'T EDIT THE UNIT FILE DIRECTORY. ALWAYS USE DROP-IN FILES.
/etc/systemd/system/mcgruber.service.d/10-cpulimits.conf
```shell
[Service]
LimitCPU=30
```
After adding or editing this file, the ```systemctl daemon-reload``` command can be used to inform systemd of the changes.

A full list of Limit*= settings, and their meanings, are described in the systemd.exec and setrlimit man pages.

##Using Control Groups(cgroups)
The linux kernel offers a mechanism for controlling resources in a fine-grained way called Linux control groups (cgroups). When using cgroups, resources are placed in controllers representing the type of resource; for example, cpu for CPU time, memory for memory usage, and blkio for disk I/O.

Inside a cgroup, resources are shared equally, but different limits and/or weights can be set on different cgroups, as long as they don't exceed the limits of the parent cgroup. When a new cgroup is created it normally inherits the limits set on its parent cgroup, unless explicity overridden.

###cgroups and systemd
By default, systemd will subdivied the cpu, cpuacct, blkio, and memory cgroups into three equal parts (called slices by systemd): ```system``` for system services and daemons, ```machine``` for virtual machines, and ```user``` for user sessions.

Administratos can create extra *.slice units, with their own limits, and assign services to these slices. It is also possible to create slices as children of another slice by naming them ```<parent>-<child>.slice```.


* system.slice - All services started by systemd will be put into a new child group of this slice by default

* user.slice - A new child slice is created in this slice for every user that logs into the system.

* machine.slice - Virutal machines managed by libvirt will be automatically assigned a new child slice of this slice.

###Inspecting cgroups
```systemd``` ships with two tools to inspect the current cgroup/slice layout: systemd-cgtop and systemc-cgls.

A single command can also be run inside a systemd slice by executing it with [systemd-run](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#systemd-run) command, and using the --slice=option.


###Managing systemd cgroup settings

1. Enabling CPU, memory, and/or block I/O accounting for a service or a slice

2. Placing resource limits on an individual service or slice

3. 

#### Enabling accounting
Add these the the [Service] stanza in the unit files, or [Service] stanza in slice files:
```shell
[Service]
CPUAccounting=true
MemoryAccounting=true
BlockIOAccounting=true
```

The easiest way to add these settings to a unit is to use a drop-in file. Drop-in files can be configured by creating a directory under /etc/systemd/system/ named after the unit to be configured wiht .d appended. For example, the drop-in directory for sshd.service would be /etc/systemd/system/sshd.service.d/.

####Enforcing limits
Create a stanza in the unit files:
```shell
[System]
CPUShares=512
MemoryLimit=1G
BlockIO*=
```

A full list of the directives can be found in [man systemd.resource-control](http://www.freedesktop.org/software/systemd/man/systemd.resource-control.html)

####Running services in a custom slice
Ro run a service in a different slice, the setting Slice=other.slice can be used inside the [Service] block, where other.slice equls the name of a custom slice.
```shell
[Service]
Slice=other.slice
```

To create a slice as a child of another slice(inheriting all settings from the parent unless explicity overridden), the slice should be named parent-child.slice

#Chapter 5 - Hardware Profiling
##Hardware Resources
###CPU
###Memory
###Storage
###Networking

##Reviewing kernel messages
```shell
cat /proc/kmsg
cat /var/log/dmesg
```
###Exploring dmesg output
```shell
less /var/log/dmesg
```
##Determining CPU information
```shell
lscpu
lscpu -p
x86info
getconf -a
```
Output of lscpu is explained on pg. 112

##Retrieving SMBIOS/DMI information
dmidecode utility provided by the dmidecode package probes the local SYstem Management BIOS (SMBIOS) and Desktop Management Interface.
```shell
dmidecode
```
##Retrieving peripheral information
```shell
lspci
lsusb
lsusb -vv
```

##Generating a system profile with sosreport
```shell
sosreport -l
```
/etc/sos.conf
```shell
[plugins]
disable = rpm, grub2
```

##Profiling Storage
###Storage Hardware
####Magnetic tapes
####Magnetic hard disks
####Optical discs
####Flash drives
####Punch cards
###Profiling Storage
####Profiling storage with bonnie++
####Profiling storage with zcav
```shell
zcav -c1 -lvda1.zcav -f/dev/vda1
```
####Plotting zcav data with gunplot
####Different forms of throughput


#Chapter 6 - Software Profiling
Multitasking -
Process Scheduler -

##O(1) Scheduler
Run queue -
Expiered queue -
It makes it difficult to measure what the real priority of a process is.

##Completely Fair Scheduler(CFS)
Uses a red-black tree based on virtual time. This virtual time is based on the time waiting to run and the number of processes vying for CPU time, as well as the priority of the process(es). The process with the most virtual time, which is the longest time waiting for the CPU, gets to use the CPU. As it uses CPU cycles, its virtual time decreases. Once the process no longer has the most virtual time, it is pre-empted and the process with the most virtual time then runs. 

Kernel tunables for CFS available in /proc/sys/kernel:

* sched_latency_ns - THe epoch duration in nanoseconds. The scheduler will run each task on the queue once during the epoch.

* sched_min_granulairty_ns - The granularity of th epoch in nanoseconds. It must be smaller than ```sched_latency_ns```.

* sched_migration_cost_ns - The cost of migrating tasks amoung CPUs. If the real run time of the task is less than this value, then the scheduler will assume that is is still in cache and willt ry to avoid moving the task to another CPU.

* sched_rt_runtime_us - The maximum CPU time (in microseconds) that can be used by all real-time tasks in a ```sched_rt_period_us``` time period. ```sched_rt_period_us-sched_rt_runtime_us``` is reserved for non realtime tasks

* sched_rt_period_us - The time slice in which ```sched_rt_runtime_us is calculated

##Controlling CPU scheduling

### Changing CPU scheduling on the command-line with systemd managed services.

```shell
#This makes a drop-in file for the respective service
systemctl set-property httpd.service CPUShares=2048

#This only changes the tunable at runtime, it's not persistant across reboots.
systemctl set-property --runtime httpd.service CPUShares=2048

#Show the value of the specific tunable.
systemctl show -p CPUShares system.slice
```

##Real-time CPU scheduling

TWo real-time scheduling policies:

1. SCHED_RR - a round-robin scheduling policy. Processes at the same priority level using the round-robin scheduling policy are only allowed to run for a maximum time quantum.

2. SCHED_FIFO - a first-in first-out schedule. It runs until it is blocked by I/O, calls ```sched_yield```, or is pre-empted by a higher-priority process.

A system admin can prevent real-time tasks from taking all CPU time with two sysctl tunables:

* ```kernel.sched_rt_period_us```: the time period in microseconds that represents the CPU allocation time frame.

* ```kernel.sched_rt_runtime_us```: The share of the time frame that can be allocated for real time. THe default for this setting on a bare-metal installation of Red Hat Enterprise LInux 7 is 950000, which means that 0.95 seconds of the 1 second period can be allocated to real-time scheduled processes. The remaining 0.05 second is used by SCHED_OTHER. This setting prevents the system from allocation all CPU time to real-time processes, which would cause the system to be unresponsive. A setting of -1 disables reservation of time for real-time and non-real-time processes.

###Non-real-time scheduling policies

1. SCHED_NORMAL - The standard round-robin style time-sharing policy

2. SCHED_BATCH - is tuned for batch-style process execution. It does not pre-empt nearly as often as SCHED_NORMAL does, so the tasks run longer and make better use of caches.

3. SCHED_IDLE - is for running very low priority applications. The priority of processes running with SCHED_IDLE is lower than a process running with nice 19

###Setting process priority with nice and renice

```shell
nice -n -5 sha1sum /dev/zero
renice 19 2190
```

###Changing process schedulers
The chrt command may start a program with a specified scheduler and a given priority. It may also change the scheduler and optionally the priority on an already running process.
```shell
chrt [scheduler] priority command
```

###The effects of different real-time policies running concurrently
You can change the scheduler policy with the following command:
```shell
chrt -p -f 10 $(cat /var/run/sshd.pid)
```

##Tracing System and Library Calls

* system call - A kernel-provided function that is called from a program. Always operating system specific.

* library call - A function provided by a library.


###System calls and library calls

###Tracing System calls

```shell
strace uname
strace -p 3124

#Display counts instead of detailed function call information
strace -c uname

#With the -f option, strace follows child processes created by the fork() system call
strace -fc elinks -dump http://classroom.example.com/pub

#With the -e option, the output can be filtered
strace -e open -c uname
```


###Tracing library calls
The ```ltrace``` command displays calls made to library functions by a process.

```shell
ltrace uname
ltrace -p 2443
ltrace -c uname
ltrace -f elinks -dump http://classroom.example.com/pub
```

## Profiling CPU Cache Usage

###Cache architecture
Cache memory is organized into lines. Each line of cache can be used to cache a specific location in memory.

The cache controller first checks to see if the requested address is in cache and will satisfy the processor's request from there if it is. This is referred to as a cache hit. If the requested memory is not in cache, then a cache miss occurs and the requested location must be read from main memory and brought into cache. This is referred to as a cache-line fill.

Cache memory can be configured as either write-through or write-back. IF write-through caching is enabled, then when a particular line of cache memory isupdated, the corresponding location in main memory must be updated as well. IF write-back caching is enabled, a write to a cache line does not get written back to main emmory until the cache line is deallocated.

* Direct mapped cache is the least expensive type of cache memory. Each line of direct mapped cache can only cache a specific location in main memory

* Fully associative cache memory is the most flexible type of cache memory and consequently the most expensive, as it requires the most cicuitry to implement. A fully associative cache memory line can cache any location in main memory.

* Set associative cache memory is uually referred to as n-way set associative, where n is some power of 2. Set associative cache memory allows a memory location to be cached into any one of n lines of cache.

###Locality of reference
Programs tend to exhibit certain patterns when accessing data. A program accessing memory location X is most likely to want to acess memory location X+1 in the next few cycles of its execution. This behavior of a program is referred to as spatial locality of reference. This is one of the reasons that it si more efficient to move data from disk to memory in pages rather than one byte at a time. Another type of pattern, temporal locality of reference, occurs when a program accesses the same memory location repeatedly during a time period.

###Profiling cache usage with valgrind
```shell
yum install valgrind cache-lab
valgrind --tool=cachegrind cache1
```


###Profilling cache misses with perf
```shell
perf list
perf stat -e cache-misses cache1
```


# Chapter 7 - Using Systemtap

##Installing systemtap

You must install these packages:

* kernel-debuginfo

* kernel-devel

* systemtap

Verify works just fine:
```shell
stap -e 'probe begin { printf("Hello World"!\n") exit()}'
```

##Using stap to run SystemTap scripts

##Compiling SYstemtap programs to portable kernel modules
```shell
stap -p 4 -v -m syscalls_by_proc /usr/share/doc/systemtap-client-*/examples/process/syscalls_by_proc.stp
staprun syscalls_by_proc
```



##Running SystemTap programs as a non-root user - pg. 178
Modules (.ko files) must be plublished in /usr/lib/modules/$(uname -r)/systemtap in order for non-root users to have access to them.
User must also be in the ```stapusr``` system group.

##Deploying SystemTap Instrumentation Modules - pg. 183

# Small File Tuning - Chapter 8
##Analyzing a Small File Workload

Disk Schedular algorithms

* noop -

* deadline -

* cfg -


## Selecting a File System

* XFS - In Glossary

* ext4 - In Glossary

###Benchmarking file system performance
For benchmarking results to be relevant, they must be obtained using benchmarking routines that simulate real-world workloads at the file system layer.

#Tuning the server for large memory workload - Chapter 9

##Memory Management
A normal page size is 4KiB.

Processes do not address physical memory directly. Instead, each process has a virtual address space. When a process is allocated memory, the physical address of a page frame is mapped to one of the process's virtual addresses. From the perspective of a process, it has a private memory space and it can only see physical page frames that have been mapped into one of its virtual addresses.
```shell
ps up <pid>
```

##Page tables and the TLB
Since each process maintains its own virtual address space, each process needs its own table of mapings of the virtual addresses of pages to the physical addresses of page frames in RAM. On a 64-bit system, the high 52 bits (63 to 12) are the page number. However, on processors in production at the time of writing, the highest 16 bits of the virtual address are reserved, which causes a large block of virtual addresses to be unavailable for use by processes. This was done because it conserves certain resources on current hardware designs. 

Major and minor page faults - see glossary



##Process memory
In RHEL7, memory management settings are moved from the process level to the application level through binding the system of cgroup hierarchies with the systemd unit tree.

##Finding memory Leaks
To identify memory leaks, use tools like ps, top, free, sar -r or sar -R, or use dedicated tools like valgrind.


See commands page for valgrind command examples.

##Tuning swap
vmstat can be used to identify swapping. The critical columns for identifing swaps are ```si``` and ```so```.

###System memory and page cache
The kernel uses most unallocated memory as a cache to store data being read from or written to disk. The next time that data is needed, it can be fetched from RAM rather than the disk.

###Swappiness
When the kernel wants to free a page of memory, it has to evaluate the tradeoff between two choices. It can swap a page out from process memory, or it can drop a page from the page cache. In order to make this decision, the kernel will perform the following calculation:

[swap_tendency](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/formulas.md#swap-tendency)

Tuning [vm.swappiness](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#sysctl-tunables) can influence a system serverly. Set [vm.swappiness](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#sysctl-tunables) to 100 and the system will almost always prefer to swap out pages over reclaiming a page from the page cache. This will use more memory for pache caches, which can greatly increase performance for an I/O-heavy workload. Setting it to 1, on the other hand, will force the s ystem to swap as little as possible.

###Optimizing swap spaces
When mulitple swap spaces are in use, the mount option [pri=value](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#swapon) can be used to specify the priority of use for each space. Swap spaces with a higher ```pri``` value will be filled upf irst before moving on the one with a lower priority. When mulitple swap spaces are activated with the same priority, they will be used in a round-robin fashion.

##Managing Memory Reclamation
Physical memory needs to be reclaimed from time to time to stop memory from fillingup, rendering a system unusable. Memory can be in different states:

* [Free](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/glossary.md#states-of-a-memory-page)

* [Inactive clean](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/glossary.md#states-of-a-memory-page)

* [Inactive Diry](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/glossary.md#states-of-a-memory-page)

* [Active](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/glossary.md#states-of-a-memory-page)

Display /proc/meminfo to get a systemwide overview of memory allocations.

For a per-process view, use /proc/PID/smaps.

Dirty pages must be written to disk to keep memory from filling up wtih pages that cannot be freed. Writing out dirty pages to disk is handled by Per-BDI flush threads. Per-BDI flush will show up in the process list as flush-MAJOR:MINOR.

See [sysctl tunables](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#controling-per-bdi-flush-threads-for-writing-dirty-pages-to-disk) for controling when the per-BDI flush threads start writing data to disk.

Setting lower ratios results in more frequent but shorter writes, suited for an interactive system, while setting hiegher raios will result in fewer but bigger writes, causing less overhead in total but ppossibly resulting in higher response times for interactive applications.

###Out-of-memory handling and the "OOM Killer"
When a minor page fault happens, but there are no free pages avilable, the kernel tries to relaim emory to satisfy the request. If it cannot reclaim sufficient memory in a timely manner, an out-of-memory condition occurs.

To determine which process the OOM killer should kill, the kernel keeps a running badness score for every process, which can be viewed in /proc/PID/oom_score. The higher the score, the more likely the process will be killed by the OOM killer.

[More OOM Tunables](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#oom-tunables)

###Memory zones and OOM
It is possible for the system to be in an out-of-memory condition while ```free``` still reports memory available. This is because sometimes memory is needed from a particular memory zone that is not available.

One way to observe memory consumption on a per-zone basis is by looking in the /proc/buddyinfo file. The memory management system uses a "buddy allocator" to organize free memory into contiguous chunks.

##Managin Non-uniform Memory Access
On a NUMA system, system memory is divided into zones that are connected directly to particular CPUs or sockets.

To see how a system is divided into nodes, use the [numactl --hardware](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#numactl) command from the numactl package.


#Tuning for a cpu-intensive workload - Chapter 10

##Configuring CPU scheduler wakeups
In normal operation, CFS tries to give each process an even chunk of CPU when there are multiple processes in state RUNNABLE. It does this by keeping rack of how much CPU each process has used. The scheduler wakes up every [kernel.sched_min_granularity_ns](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#configuring-cpu-scheduler-wakeups) nanoseconds ( a nonsecond is 10^9 seconds, or one-billionth of a second). When the scheduler wakes up, it checks if the current running process is the one that has had the least amount of CPU so far. If it is, the process is allowed to keep on running. If the process is not the one that has had the least amount of CPU time, it will be kicked off the CPU(pre-empted). For batch processing serversk, like compute nodes, it might be useful to increase the kernel.sched_min_granularity_ns systctl so that the scheduler wakes up less often.

##Configuring maximum CPU usage
The CFS scheduler also has the idea of scheduler groups implemented. Using scheduler groups allows processes to be no longer seen as equals but rather process groups, in this case implemented using cgroups with the cpu controller. Each group is assigned a weight using the [cpu.shares]() tunable inside the group. The default weight for the / cgroup and any new cgroups created is 1024.




##Pinning Processes
Sometimes it is desirable to limit on which CPU(s) a process is allowed to run. systemd offers a convenient way limiting the CPUs available to a service using the [CPUAffinity=](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#drop-in-tunables) setting inside a [Service] block of a unit definition. This setting takes a separated list of CPU indexes the service is allowed to run on, with the first CPU being index 0, the second index 1, etc..

A scalable way to limit the available CPUs and memory zones is by using a cpuset cgroup.

* [cpuset.cpus](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#cpuset-cgroup-tunables)

* [cpuset.mems](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#cpuset-cgroup-tunables)

* [cpuset.{cpu,memory}_exclusive](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#cpuset-cgroup-tunables)

##Balancing Interrupts
Since an interrupt needs to be handled when it occurs, a CPU will need to be chosen on which to execute the corresponding code. On a single CPU system the choice is easy, but when there are multiple available CPUs, the choise of which CPU to execute the choice of which CPU to execute the code on can become interesting.

The file /proc/interrupts details how often specific interrupts have been handled on a specific CPU.

To determine on which CPUs the kernel will execute a given interrupt-handler, the kernel will look at the file /proc/irq/number/smp_affinity. This file has a bitmask in it, represented as a hexadecimal number. For every CPU that an interrupt is allowed to be handled on, the corresponding bit will be set to 1; the bit will be set to 0 if the kernel is not allowed to use said CPU for that interrupt.

To calculate the value associated with a certain CPU, use the formula: [value=2^n](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/formulas.md#balancing-interrupts), where n is the CPU number, starting at 0 for the first CPU.

Alternatively, the file ```proc/irq/number/smp_affinity_list``` can be used as well. This file takes a comma separated list of CPU indexes (starting at 0) that can be used to handle the interrupt.

The purpose of irqbalance is to adjust the smp_affinity of all interrupts every 10 seconds so that an interrupt handler has the highest chance of getting cache hits when executing. On single-core systems and dual-core systems that share their L2 cache, irqbalance will do nothing.

In ```/etc/sysconfig/irqbalance```, there are two settings that can be made to influence the behavior of the irqbalance daemon.

* [IRQBALANCE_ONESHOT](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#irqbalance-tunables)

* [IRQBALANCE_BANNED_CPUS](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#irqbalance-tunables)

##Employing Real-time Scheduling
So far, it has been assumed processes are being scheduled in the default scheduler policy: SCHED_NORMAL(also known as SCHED_OTHER). Processes can also be put under different scheduler policies, changing the way they will be preempted and allocated time slots on the CPU.

* [SCHED_NORMAL or SCHED_OTHER](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/glossary.md#cpu-schduling-policies)

* [SCHED_BATCH](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/glossary.md#cpu-schduling-policies)

* [SCHED_IDLE](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/glossary.md#cpu-schduling-policies)

* [SCHED_FIFO](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/glossary.md#cpu-schduling-policies)

* [SCHED_RR](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/glossary.md#cpu-schduling-policies)


#Tuning a file server - Chapter 11 - pg. 285

##Selecting a tuned profile for a file server
The throughput-performance profile, and the network-throughput profile that inherits its properties, both increase the default [readahead](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#tuned) setting for block devices by a factor of 32. This setting influences how much data the kernel will attempt to read ahead when it sees contigous requests come in.

The [blockdev](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#blockdev) command can be used to read and write the ```read_ahead_kb``` tunable. Use the ```--getra``` and ```setra``` options to blockdev to get and set the readahead values respectively.


##File System Performance
###File fragmentation
Both the default file system in RHEL7, XFS, and its predecessor, ext4, attempt to minimize fragmentation by pre-allocating extra data blocks on disk when a file is being written. If the extra blocks have not yet been used when the file is closed, they are marked as free again.

In order to make it more likely that large areas of contiguous free space are always available, the ext4 file system also sets aside reserved space, normally given as a percentage of total file system space or a number of file system blocks.

Fragmentation in a file system can be reported in multiple ways. FOr the XFS file system, the [xfs_db](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#xfs_db) command can report a fragmentation factor.

With the ext4 file system, fragmentation can be reported as a percentage of fragmented files with the [fsck.ext4](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#fsck) command.

The [filefrag](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#filefrag) utility can be used to report the number of extents (contiguous space on disk) used by a particular file on either the XFS or ext4 file system.

The [xfs_bmap](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#xfs_bmap) utility can be used to generate a fragmentation report for a particular file on XFS file systems only.

On ext4 file systems, the number of free file system extents of a given size can be reported by the [e2freefrag](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#e2freefrag) command.

Both XFS and ext4 provide tools for performing online defragmentation. The [xfs_fsr](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#xfs_fsr) and [e4defrag](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#e4defrag) utilities are available for the XFS and ext4 file systems, respectively. Both tools offer multiple defragmentation methods

###Journal placement
A journal speeds file system recovery by being a sort of logbook for the file system. Whenever a change is about ot be made to the file system, the transaction will be recorded in the journal. After the operation is completed, the journal entry is removed.

The ext3/ext4 journal can work in three different modes, chosen by passing the data=mode option to the file system at mount time. The default mode is ```ordered```. Thre three modes are:

* [ordered](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/glossary.md#ext3ext4-journal-modes)

* [writeback](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/glossary.md#ext3ext4-journal-modes)

* [journal](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/glossary.md#ext3ext4-journal-modes)

To enable or disable barriers at mount time on XFS file system, use either the barrier or the nobarrier option, respectively. On devices with a battery-backed cache, barriers can be disabled for a performance improvement while still guaranteeing data integrity.

By default, when XFS and ext4 file systems are created, their journals are placed on the same block device as the file system data. With certian workloads, such as those involving a large number of random writes, there may be disk contention between the write operations and their accompanying journal updates.One way to imporve journal performance is to place the journal on a separate device from the one that contains the file system.

To create a new XFS file system with an external journal, use the ```logdev``` log section option
```shell
mkfs -t xfs -l logdev=/dev/ssd1 /dev/sdc1
mount -o logdev=/dev/sdd1 /dev/sdc1 /mnt
```

###Stripe unit and width
What typically has a much greater performance impart is how file system metadata is aligned on disk when the disk in question is striped array (RAID 0, RAID 4, RAID 5, RAID 6). If a file system is not laid out to match the RAID loayout, it could happen that a file system metadata block is spread out over two disks, causing two disk requests instead of one, or that all of the metadata ends up on one individual disk, causing that disk to become a hot spot.

To counter this, lay out your file system during creation to match the layout of the RAID array that it resides on. For XFS file systems, optimization for striped allocation requires the following information:

* The chunk size of the RAID array

* The number of disks in the RAID array

* The number of parity disks in the RAID array

Subtracing the number of parity disks from the total number of disks will provide the number of data disks in the array.

The ```su``` and ```sw``` data section options are used to specify the RAID array chunk size and the number of data disks, repectively.
```shell
mkfs -t xfs -d su=64k,sw=4 /dev/san/lun1
```

For ext4 file systems, the following information is required for file system alignment:

* The chunk size of the RAID array

* The number of disks in the RAID array

* The number of parity disks in the RAID array.

This information is then used to calulate the file system [stride](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/formulas.md#stripe-unit-and-width) and [stripe-width](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/formulas.md#stripe-unit-and-width).

These numbers are provided to mkfs at file system creation
```shell
mkfs -t ext4 -E stride=16,stripe-width=64 /dev/san/lun1
```

##Tuning network performance.
There are a variety of tools for measuring network throughput. One of the newer and more extensive tools is [qperf](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#qperf). When using an Ehternet network, qperf can measure TCP, UDP, RDS, SCTP, and SDP socket throughputs and latencies.

[qperf](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#qperf) needs to run on two machines. One(the listener) runs qperf without any options. The other (the sender) invokes qperf with the name of the first host as the first argument, followed by the test options.

##Tuning network queues
The following outline shows the steps of network transmission:

* Data is written to a socket (a file-like object) and is then put in the transmit buffer.

* The kernel encapsulates the data into a protocol data unit(PDU)

* The PDUs move to the per-device transmit queue

* The network device driver copies the PDU from the head of the tranmit queue to the NIC

* The NIC sends the data and raises an interrupt when transmitted

The following outline show the steps of network reception

* The NIC receives a frame and uses DMA to copy the frame into the receive buffer.

* The NIC rasies a hard interrupt.

* The kernel handles the hard interrupt and schedules a soft interrupt to handle the packet

* The soft interrupt is handled and moves the packet to the IP layer.

* If the packet is destined for local delivary, the PDU will be decapsulated and put in a socket receive buffer. If a process was waiting on this socket, it will now process the data from the receive buffer.

These buffer sizes can be manipulated with some [sysctl tunables](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#buffer-size-sysctl-tunables)

To calculate the buffer sizes needed for maximum throughput, determine the amount of time that it takes for a packet to travel from the local system to the remote system. This can be used to calculate the amount of data that can be sent before the first packet has been received and acknowledged. First, figure out the round trip time of a packet, the time it takes for a packet to be setn from the local machine and returned from the remote machine. The ```ping``` command can be used to find the average round trip time.

This calculation is called [bandwidth delay product](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/formulas.md#bandwidth-delay-product), or BDP for short.

If the sysctl [net.ipv4.tcp_window_scaling](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#window-scaling) parameter is set to 1, the kernel will attempt to negotiate window scaling. Window scaling will be disabled if this value is 0.

A method to reduce over head is to increase the size of the packets that can be sent (the MTU). When increasing the MTU above the Ethernet standard or 1500 bytes, the resulting packets are called jumbo frames.

Before configuring a system to use jumbo frames with a specific NIC, must make sure that every piece of networking equipment on the network supports jumbo frames, including bot not limited to the NICs, switches, and routers. The official maximum size for a jumbo frame is 9000 bytes, but some equipment supports even larger frames.

To configure a higher MTU, add the following line to ```/etc/sysconfig/network-sciprts/ifcfg-name```:
```shell
MTU=size
```

#Tuning a database server - Chapter 12

##System memory

##Disk storage

##Networking

##Managing Inter-process Communication - pg. 338
Semaphores allow two or more processes to coordinate access to shared resources. Message queues allow processes to cooperatively function by exchanging messages. System administrators can set limits on the number of SysV IPC resources available to processes.

Current SysV IPC resource limits can be obtained by running [ipcs](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#ipcs)

The SysV IPC mechanisms are tuned using entries in [/proc/sys/kernel/](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#sysv-ipc-tunables).


/dev/shm - temporary storage. Will be lost if a power failure occurs.

##Managing Huge Pages
The Linux kernel supports large-sized memory pages through the huge pages mechanism(sometimes known as bigpages, largepages or the hugetlbfs file system). Most processor architectures support multiple page sizes.

The number of translation lookaside buffer(TLB) entries for a process is fixed, but with larger page size, the TLB space for a processor becomes correspondingly larger. Having fewer TLB entries that point to more memory means that a TLB hit is more likely to occur.

Look in the /proc/meminfo file to dtermine the size of a huge page on a particular system.
```shell
cat /proc/meminfo
...
Hugepagesize: 2048kB
...
```

In order to allocate huge memory pages, set the total number of huge pages required using the [nm.nr_hugepages](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#huge-pages-tunables) sysctl parameter.

In order to use the large pages, processes must request them using either the mmap system call or the shmat and shmget system calls. If mmap is used, then the large pages must be made avilable via the hugetlbfs file system:
```shell
mkdir /largefile
mount -t hugetlbfs none /largefile

or in /etc/fstab
hugetlbfs /hugepages hugetlbfs defaults
```
###Transparent huge pages
RHEL6.2 introduced kernel functionality that supports the creation and management of huge memory pages without explicit developer or system administrator intervention. This feature is called transparent huge pages(THP). THP are enabled by default and they are used to map all of the kernel adress space to a single huge page to reduce TLB pressure. They are also used to map anonymous memory regions used by applications to allocate dynamic memory.

THP differ from standard huge pages in that they are allocated and managed by the kernel automatically when they are enabled. They can also be swapped out of memory, unlike standard huge pages.

THP tunables are found in /sys tree under the [/sys/kernel/mm/transparent_hugepage](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#transparent-huge-pages-tunables) directory.

Transparent huge pages are not recommended for use with datbase workloads; instead, if huge pages are desired, they should be preallocated with the [sysctl vm.nr_hugepages](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#huge-pages-tunables) parameter.
##Overcomitting Memory
###Tuning overcommit
Many applications request to allocate more memory than they will use. Other applications require the kernel to allocate more memory for a program than is available on the system. One way to address these situation is to manage the machine's memory overcommitment policy. A machine's memory overcommitment policy is set using the [vm.overcommit_memory](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#overcommit-tunables) tunable.

The ```Committed_AS``` field in ```/proc/meminfo``` shows an estimate of how much RAM is required to avoid an out-of-memory(OOM) condition for the current workload on a system.

The default for memory overcommitment on RHEL6 and later is 0 heuristic overcommit handling. If a machine is in a situation where it has overcommitted too much and now has to deliver more memory than is possible, ther kernel OOM killer thread is invoked and processes are killed.
##Tuning swappiness
[vm.swappiness](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#sysctl-tunables)
##Tuning cached page writes
Beyond moving the I/O elevator to deadline, which would give preference to disk reads, an administrator can also change settings that control how the system manages cached data of file writes. There are [sysctl tunables](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#controling-per-bdi-flush-threads-for-writing-dirty-pages-to-disk) that control writing of cached file data.

#Tuning Power usage - Chapter 13 - pg.359
##Tuning and Profiling Power Usage
[powertop]()
##Tuning power usage
[powertop2tuned](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#powertop2tuned)

#Tuning for virtualization - Chapter 14 - pg.379

##Kernel Samepage Merging(KSM)
When guests are running identical operating systems and/or workloads, there is a high chance that many memory pages will have the exact same content. Using Kernel Samepage Merging(KSM), total memory usage can be reduced by mergin those identical pages into one memory page. When a guest writes to that page, it will be converted into a new, separate page on the fly.

```ksm``` can be configured manually in /sys/kernel/mm/ksm/, the following files are used to control ksm, but remember that ksmtuned will adjust [these settings](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/tunables.md#ksm-tunables) as necessary if it is running.

To tune ksm, it is oftern more useful to use the ksmtuned service than to tune by hand. To confiure the ksmtuned service, use the file ```/etc/ksmtuned.conf```. To see what ksmtuned is doing, uncomment the LOGFILE and DEBUG lines and restart ksmtuned.

##Setting limits on virtual machines
By default, libvirt will configure controller groups(cgroups) for vitual machines running on the hypervisor. Adminstrators can view the cgroup definitions by using ```lscgroup```.

libvirt provides an API for interacting with the cgroup limits assigned to individual running guests. Admins will use [virsh](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#virsh) for viewing and setting resource limits on virtual machines.

##Pinning virutal machines to CPUs
libvirt also provides a mechanism for pinning virtual machine vCPUs to physical hypervisor CPUs. Like with setting control group limits, administrators use the [virsh](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/commands.md#virsh) command.
