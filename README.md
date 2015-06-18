# RHCA-Performance-Study-Guide



# Chapter 2 - Collecting, Graphing, and Interpreting Data

## Units and Unit Conversions

## Profiling Tools

###VMstat

###Sar

###IOstat

###MPstat


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

|Direcotry|Description|
|/etc/proc/sys/dev | Contains tunables for devices such as RAID devices, CD-ROMs, SCSI devies, and one or more prallel ports|
|/etc/proc/sys/fs | Holds file-system related tunables |
|/etc/proc/sys/kernel | COntains tunables that change how the kernel works internally|
|/etc/proc/sys/net | Contains tunables that change network-related settings.|
|/etc/proc/sys/vm | Contains tunables which change the virtual memory management of the kernel.|
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
Some useful commands:
```shell
lsmod - list kernel modules
modinfo <module> - list module info
modinfo -p <module> - list module parameters
```
The /sys directory contains information provided by the kernel about the state of the system. For example, the /sys/module/<module-name>/parameters/<parameter-name>.


To permanently set the same value every time the module is loaded, create a special configuration file in /etc/modprob.d/<name>.conf:
```shell
options <module-name> <parameter>=<value>
```

You can also pass kernel module paramters at load time. 
```shell
modprobe usb_storage delay_use=2
```

cat /sys/module/usb_storage/parameters/dealy_use
/sys is access to the kernel itself.
You can't change the /sys parameters on the fly when a module is already loaded. You need to tune them at load time. 
modprobe -r usb_storage
lsmod | grep usb

To change parameters at load time:
```shell
modprobe usb_storage dealy_use=5
```
This is a good way to test, we still need to make it perminent.


##Installing and Enabling tuned
```shell
systemctl enable tuned
systemctl start tuned
```
tuned on RHEL7 ships with 10 tunining profiles.
For static tuning, mostly predefined sysctl and /sys settings are appliedt to the system.
Profiles can be switched based on the current system time.
Dynamic tuning.
Some tuned profiles activate helper programs that montior the activity of selected subsystems.
/etc/tuned/tuned-main.conf is the main configuration file.

To configure dynamic tuning, change the dynamic_tuning setting to 1 in /etc/tuned/tuned_main.conf

Useful Commands:
```shell
tuned-adm active
tuned-adm list
tuned-adm profile <profile-name>
tuned-adm off
```

##Creating Custom tuned Profiles
file://usr/share/docs in a web browser to view docs.

/usr/lib/tuned/<profile>/tuned.conf for existing profiles

/etc/tuned/ for cusotom profiles

just make a directory there, and make a tuned.conf file inside that directory.

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

###cgroups and systemd

* system.slice - All services started by systemd will be put into a new child group of this slice by default

* user.slice - A new child slice is created in this slice for every user that logs into the system.

* machine.slice - Virutal machines managed by libvirt will be automatically assigned a new child slice of this slice.

Three names slices are created by default, but admins can create extra *.slice units, with their own limits, and assign services to these slices. It is also possible to create slices as children of another slice by naming them <parent>.<child>.slice. This ensures that the new slice will be created inside the <parent>.slice slice.

###Inspecting cgroups
```shell
systemd-cgtop
systemd-cgls
systemd-run --slice=example.slice sleep 10d
```

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

A full list of the directives can be found in ```man systemd.resource-control```

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

##Deploying SystemTap Instrumentation Modules - pg. 183

# Small File Tuning - Chapter 8
##Analyzing a Small File Workload

Disk Schedular algorithms

* noop -

* deadline -

* cfg -


## Selecting a File System

* XFS

* ext4

###Benchmarking file system performance


##Tuning for a Mail Server - pg 205

###Hardware

###Operating System

###Software

#Tuning the server for large memory workload - Chapter 9

##Memory Management

```shell
ps up <pid>
```

##Page tables and the TLB

##Process memory

##Finding memory Leaks
```shell
yum install -y valgrind bigmem
valgrind --tool=memcheck bigmem 256
valgrind --tool=memcheck bigmem -v 256

##Tuning swap

###System memory and page cache

###Swappiness

###Optimizing swap spaces

##Managing Memory Reclamation

###Out-of-memory handling and the "OOM Killer"

##Managin Non-uniform Memory Access
```shell
numactl --interleave=all bigdatabase
numactl --cpunodebind=0 --membind=0,1
numactl --preferred=1
numactl --show
numactl --localalloc /dev/shm/file
```


#Tuning for a cpu-intensive workload - Chapter 10

##Configuring maximum CPU usage



##Pinning Processes
```shell
CPUAffinity=""
```

##Balancing Interrupts


##Employing Real-time Scheduling


#Tuning a file server - Chapter 11 - pg. 285














