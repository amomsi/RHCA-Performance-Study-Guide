#Tunables


##Drop-in tunables
```shell
#Adjust the OOM Score, lower values means it will not be eligable for killing
[Service]
OOMScoreAdjust=<value>

#Configure the CPU scheduling policy for a process
[Service]
CPUSchedulingPolicy=batch

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
#Adjust core recieving memory buffer max
net.core.rmem_max=<value in bytes>

#Adjust tcp recieving memory buffer
net.ipv4.tcp_rmem="<min-bytes> <default-bytes> <max-bytes>"

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


##OOM tunables
```shell
#Can take values from -17 to 15 where 0 means no change, -17 means immunity(never kill) and any other value will be used to modify oom_socre by mulitplying oom_score with 2^oom_adj. Therefore, setting a positive oom_adj value makes a process more likely to be killed, while setting a negative value reduces the chances of being terminated by the kernel.
echo value > /proc/PID/oom_adj

#Values put into oom_score_adj are also added to the oom_score. However, the mechanism for using oom_score_adj is different than oom_adj. oom_score_adj can be set to values ranging from -1000 to 1000. When a value is put into oom_score_adj that value is added to the oom_score.
echo value > /proc/PID/oom_score_adj
```