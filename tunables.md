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


```