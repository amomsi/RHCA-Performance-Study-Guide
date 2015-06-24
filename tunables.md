#Tunables


##Tuned
```shell
#Configure the influence on how much data the kernel will attempt to read ahead when it sees contigous requests come in.
[disk]
readahead=number
```

##Drop-in tunables
```shell
#Adjust the OOM Score, lower values means it will not be eligable for killing
[Service]
OOMScoreAdjust=<value>

#Configure the CPU scheduling policy for a process
[Service]
CPUSchedulingPolicy=batch

#Sets the CPU scheduling priority for executed processes.
[Service]
CPUSchedulingPriority=

#Designate what CPU a process should run on
[Service]
CPUAffinity=<CPU#>

#The amount of memory usable by a systemd service unit
[Service]
MemoryLimit=<Value><Size>

#Configuring maximum CPU usage
[Service|Slice]
CPUShares=1024

```
##sysctl tunables
```shell

#Adjust semaphore properties
kernel.sem="<max sysV semaphores per semaphore array> <max sema systemwide> <max operations allowed per sema system call> <max sema arrays>"

#Number of maximum huge pages on a system
vm.nr_hugepages=32

#Enable tcp fast open???
net.ipv4.tcp_fastopen=<##>

#Configure the kernel to accommodate more open files and sockets
fs.file-max=<value>
kernel.threads-max=<value>

#Configure swap tendency or influence
vm.swappiness=<value>
```

###Buffer size sysctl tunables
```shell
#Adjust network core read memory buffer max
net.core.rmem_max=<value in pages>

#Adjust network core write memory buffer max
net.core.wmem_max

#Adjust tcp recieving memory buffer
net.ipv4.tcp_mem="<min-pages> <pressure-pages> <max-pages>"
net.ipv4.udp_mem="<min-pages> <pressure-pages> <max-pages>"

#The receive/send TCP socket buffers. Values are in bytes. A socket buffer will start at the size of default bytes (second value) and be automatically adjested between min(first value) and max(thrid value) based on need.
net.ipv4.tcp_rmem="<min-bytes> <pressure-bytes> <max-bytes>"
net.ipv4.tcp_wmem="<min-bytes> <pressure-bytes> <max-bytes>"
```

###Window scaling
```shell
#Attempt to negotiate window scaling. 1 for enabled, 0 for disabled.
net.ipv4.tcp_window_scaling=[1|0]
```

###Controling per-BDI flush threads for writing dirty pages to disk
```shell
#How old (in 1/100ths of a second) dirty data must be before it is eligible for being written out to disk. This stops the kernel from writing out the same page multiple times in quick succession just because a process modifies another byte of memory.
vm.dirty_expire_centisecs=value

#How often(in 1/100ths of a second) the kernel will wake up the flushing threads to write out data. Setting this to 0 will diable periodic writeback completely.
vm.dirty_writeback_centisecs=value

#The percentage of total system memory being diry at which the kernel will start writing out data in the background.
vm.dirty_background_ratio=value

#The percentage of total system memory being dirty at which a process generating writes will block and write out dirty pages.
vm.dirty_ratio=value
```

###Configuring CPU scheduler wakeups
```shell
#The number of nanoseconds the between each scheduler wake-up
kernel.sched_min_granularity_ns=nanoseconds

```

###SysV IPC tunables
```shell

#Specifies the maximum number of shared memory segments systemwide.
kernel.shmmi=<num>

#Specifies the total amount of shared memory, in pages, that can be used at one time on the system. ipcs -l displays this value after converting it to KiB. This should be at least kernel.shmmax/PAGE_SIZE, where PAGE_SIZE is the page size on the system (4KiB is typical). PAGE_SIZE can be looked up using getconf PAGESIZE.
kernel.shmall

#Specifies the maximum size of a shared segment that can be created, in bytes.
kernel.shmmax=<bytes>

#Specifies the maximum number of bytes in a single message queue.
kernel.msgmnb=<bytes>

#Specifies the maximum number of message queue identifiers
kernel.msgmni=<num>

#Specifies the maximum size of a message that can be passed between processes. Note that this memory cannot be swapped.
kernel.msgmax=<size>

#Contains settings for:
# The maximum number of semaphores per semaphore array.
# The maximum number of semaphores allowed systemwide.
# The maximum number of allowed operations per semaphore system call.
# The maximum number of semaphore arrays.
kernel.sem=< x x x x >
```


##OOM tunables
```shell
#Can take values from -17 to 15 where 0 means no change, -17 means immunity(never kill) and any other value will be used to modify oom_socre by mulitplying oom_score with 2^oom_adj. Therefore, setting a positive oom_adj value makes a process more likely to be killed, while setting a negative value reduces the chances of being terminated by the kernel.
echo value > /proc/PID/oom_adj

#Values put into oom_score_adj are also added to the oom_score. However, the mechanism for using oom_score_adj is different than oom_adj. oom_score_adj can be set to values ranging from -1000 to 1000. When a value is put into oom_score_adj that value is added to the oom_score.
echo value > /proc/PID/oom_score_adj
```

##cpuset cgroup tunables
```shell

#Configure the CPUs that an be used inside this cgroup.
echo 0 > /sys/fs/cgroup/cpuset/<cgroup#>/cpuset.cpus

#Configure which NUMA memory zones to use inside this cgroup.
echo 0 > /sys/fs/cgroup/cpuset/<cgroup#>/cpuset.mems
```

##irqbalance tunables
```shell
#When set to yes, irqbalance will sleep for one minute after startup, rebalance interrupts once, and then exit. This can be useful for avoiding the overhead of iqbalance waking up every 10 seconds
IRQBALANCE_ONESHOT=yes

#If set, irqbalance will not assign any interrupts to the CPUs listed. The value is a hexadecimal bitmask of all CPUs available on the system, calculated in the same way as smp_affinity. 
IRQBALANCE_BANNED_CPUS=fffffffffffffffe