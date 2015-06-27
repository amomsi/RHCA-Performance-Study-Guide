#Commands

##vmstat
```shell
#Print out the averages of various system stats since boot.
vmstat

#Continously print stats, 1 second apart.
vmstat 1
```

##sar
```shell
#Commands
yum install sysstat

#Important files and directories
##Cron job that controls how often the system collects information
/etc/cron.d/sysstat
sar

#Set LANG variable for 24hr time
LANG=C sar -q

#Read one of the log files(or any sar data file)
sar -q -f <file>

#Report the I/O and transfer rate stats
sar -b

#Report the utilization of CPU0
sar -P 0

#Report network device statistics form the current log file.
sar -n DEV
```

##mpstat
```shell
LANG=C mpstat -P 0
```

##iostat
```shell
iostat -x vda
```

##gnuplot
```shell
yum install -y gnuplot
gnuplot /tmp/uptime.
gnuplot -persist /tmp/uptime.gnuplot
```
##pcp(performance co-pilot)
```shell
yum -y install pcp-gui
systemctl start pcp
systemctl enable pcp
systemctl status pcp

pmstat -s 5
pmcollectl -c 5
pmatop
pminfo
pminfo -dt proc.nprocs
pmval -s 5 proc.nprocs
```
##modinfo
```shell
modinfo <module-name>

#Show only the parameters of a module
modinfo -p module
```

##modprobe
```
modprobe <module-name> parameter_name=value
```
##tuned
```shell
#Commands
yum install tuned

#Change tuned profile
tuned-adm profile <profile>

#List all profiles
tuned-adm list

#List active profile
tuned-adm active

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

#Permanently enable cpu accounting for services.
systemctl set-property <service> CPUAccounting=yes

#Enable CPU accounting for service at runtime
systemctl set-property --runtime <service> CPUAccounting=yes

#Setup CPUShares value on a service
systemctl set-property <service> CPUShares=<number>
```

##systemd-cgtop
```
#Inspect CPU usage
systemd-cgtop
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

#Can be used to view all threads and CPU information for a process
ps mo pid,comm,psr $(pgrep <process>)

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

```

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

##top
```shell
top
```

##chrt
```shell
#Select the SCHED_BATCH schceduling policy
chrt -b <command>

#Select the SCHED_NORMAL scheduling policy
chrt -o

#Select the SCHED_IDLE scheduling policy
chrt -i

#Select the SCHED_FIFO scheduling policy
chrt -f

#Select the SCHED_RR scheduling policy
chrt -r
```

##blockdev
```shell
blockdev --getra /dev/vda
blockdev --setra 512 /dev/vda
cat /sys/block/vda/queue/read_ahead_kb
```
##xfs_db
```shell
#Report fragmentation factor for XFS file systems
xfs_db -c frag -r /dev/vdb2
```

##fsck
```shell
#Report fragmentation as a percentage of fragmented files
fsck -fn /dev/vdb1
```

##filefrag
```shell
#report the number of extents(contiguous space on disk) used by a particular file on either the XFS or ext4 file system
firefrag -v largefile
```

##xfs_bmap
```shell
#Generate a fragmentation report for a particular file on XFS
xfs_bmap -v largefile
```

##e2freefrag
```shell
#Report the number of free file system extents on a ext4 file system
e2freefrag /dev/vgsrv/root
```

##xfs_fsr
```shell
#For XFS file systems
#Target defrag a single file
xfs_fsr -v largefile

#Deframentation on file system
xfs_fsr -v /path/to/mount/point

#Defrag on specific time period
xfs_fsr -v -t 600
```

##e4defrag
```shell
#For ext4 file systems
#Defag a single file
e4defrag -v largfile

#Defrag against file system
e4defrag -v /path/to/mount/point
e4defrag -v /dev/vdb1

#defrag a directory
e4defrag -v /home/user/tmp

#Calculate defragmentation to determine if necessary
e4degrag -c /mnt/ext4
```

##mkfs/mount
```shell
#The logdev mount option is used to specify an external journal.
mkfs -t xfs -l logdev=/dev/sdd1 /dev/sdc1
mount -o logdev=/dev/sdd1 /dev/sdc1 /mnt
```

##qperf
```shell
#The listener
qperf

#The sender
qperf desktopYIP tcp_bw udp_bw
```

##ipcs
```shell
#Get current SysV IPC resource limits
ipcs -l

#The SysV IPC resources currently being used can be displayed:
ipcs
```
##powertop2tuned
```shell
power2tuned -e -m powersave lesswatts
```

##virsh
```shell
virsh schedinfo desktop
virsh blkiotune desktop
virsh memtune desktop
virsh schedinfo desktop cpu_shares=100

#View information on which physical CPU a guest may use.
virsh vcpuinfo desktop

#Pin the desktop virtual machine's vCPU 0 to run on the hypervisor's CPU1.
virsh vcpupin desktop 0 1
virsh vcpupin desktop 0 1,3
```
