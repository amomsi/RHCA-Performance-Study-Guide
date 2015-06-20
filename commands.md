#Commands


##sar
```shell
#Commands
yum install sysstat

#Important files and directories
##Cron job that controls how often the system collects information
/etc/cron.d/sysstat
sar
```

##tuned
```shell
#Commands
yum install tuned
tuned-adm profile <profile>

#Important files and directoires
##Configuration files(Create custom tuned profiles as directories here)
/etc/tuned
```

##Limits
```shell

#Important files and directories
##Limiting users or groups to certain resources
/etc/security/limits.d/<filename>.conf
```

##systemctl
```shell
systemctl set-property httpd.service MemoryLimit=512M
systemctl daemon-reload
```

##valgrind
```shell
yum install valgrind cache-lab bigmem
valgrind --tool=cachegrind <command>

#Check if a program is leaking memory
valgrind --tool=memcheck program [program arguments]


```

##systemtap
```shell
yum install systemtap kernel-devel
debuginfo-install kernel
```

##ps
```shell
#Show the virtual and physical allocation of a specific process.
ps up <pid>

#Can be used to view major and minor page faults
ps o pid,comm,minflt,majflt <pid>

#Can be used to view virtual and physical memory allocations
ps -o pid,ppid,rss,vsz,command -p <PID>



```

##x86info
```shell
#Can be used to determine the size of the TLB buffer
x86info -c
```

##pmap
```shell
#To view how the virtual address space of a process is used
pmap <pid>
```


##swapon
```shell
#Set priority for a swap device
swapon -p 1

#Important files and directories.
/etc/fstab

```shell

##numactl
```shell
yum install numactl

#Run bigdatabase with its memory interleaved across all CPUs
numactl --interleave=all bigdatabase

#Run process on node 0 with all memory allocated on node 0 and node 1
numactl --cpunodebind=0 --membind=0,1

#set node 1 as preferred , and show the resulting state.
numactl --preferred=1; numactl --show

# Reset the policy for teh shared memory file to the default local alloc policy
numactl --localalloc /dev/shm/file
```


