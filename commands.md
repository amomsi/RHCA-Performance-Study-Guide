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
#Install package thet includes sar
yum install sysstat

#Important files and directories
##Cron job that controls how often the system collects information using sar
/etc/cron.d/sysstat
sar

#Set LANG variable for 24hr time in sar
LANG=C sar -q

#Read one of the log files(or any sar data file)
sar -q -f <file>

#Report the I/O and transfer rate stats in sar
sar -b

#Report the utilization of CPU0 in sar
sar -P 0

#Report network device statistics form the current log file in sar.
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

##[gnuplot](https://github.com/elextro/RHCA-Performance-Study-Guide/blob/master/gnuplot.md#gnuplot-example-file)
```shell
#Install gnuplot
yum install -y gnuplot
gnuplot /tmp/uptime.
gnuplot -persist /tmp/uptime.gnuplot
```
##pcp(performance co-pilot)
```shell
#Install pcp gui
yum -y install pcp-gui

#Start and Enable pcp
systemctl start pcp
systemctl enable pcp
systemctl status pcp


pmstat -s 5

#pcp command that collects data about various sub systems.
pmcollectl -c 5

#top-like output of machine stats and data using pcp
pmatop

#Print various metrics about a system
pminfo

#Get information about a certain metric on a system using pcp(performance co-pilot)
pminfo -dt proc.nprocs

#Get 5 samples stored in the pcp metric proc.nprocs
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
#Load module with a parameter set.
modprobe <module-name> parameter_name=value
```
##tuned
```shell
#Install tuned
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
#Set Unit file tunable MemoryLimit to 512M
systemctl set-property httpd.service MemoryLimit=512M

#Inform systemd of any changes that might have been made to unit files.
systemctl daemon-reload

#Permanently enable cpu accounting for services.
systemctl set-property <service> CPUAccounting=yes

#Enable CPU accounting for service at runtime
systemctl set-property --runtime <service> CPUAccounting=yes

#Setup CPUShares value on a service
systemctl set-property <service> CPUShares=<number>
```

##systemd-run
```shell
#Run a single command inside a slice.
systemd-run --slice=example.slice sleep 10d
```

##systemd-cgtop
```
#Inspect control groups.
systemd-cgtop
```

##valgrind
```shell
#Install valgrind with cachegrind and bigmem tool
yum install valgrind cache-lab bigmem

#Profile cache while running a certain command.
valgrind --tool=cachegrind <command>

#Check if a program is leaking memory
valgrind --tool=memcheck program [program arguments]


```

##systemtap
```shell

#Install systemtap
yum install systemtap kernel-devel;
yum install kernel-debuginfo;
debuginfo-install kernel;

#Compile a stap script to object code(.ko format)
stap -v p 4 -m <name of object> /usr/share/doc/systemtap-client-2.4/examples/memory/<stap-script>.stp

#The directory a systemtap complied kernel module must be in for a non-root user to run them
mkdir /lib/modules/$(uname -r)/systemtap

#Run a compiled systemtab kernel module. Non-root that are a part of the stapusr group can use this.
straprun <name-of-kernel-module>
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

#Activate swap spaces
swapon -a

#Validate the swap spaces
swapon -s

#fstab entry for a swap partition
UUID=123 swap swap pri=1 0 0
UUID=1234 swap swap pri=2 0 0



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
chrt -o <command>

#Select the SCHED_IDLE scheduling policy
chrt -i <command>

#Select the SCHED_FIFO scheduling policy
chrt -f <command>

#Select the SCHED_RR scheduling policy
chrt -r <command>
```

##blockdev
```shell

#Print readhead of a disk
blockdev --getra /dev/vda

#Set readhead of a disk
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
#Target defrag a single file for XFS file systems
xfs_fsr -v largefile

#Deframentation on XFS file system
xfs_fsr -v /path/to/mount/point

#Defrag on specific time period on an XFS file system
xfs_fsr -v -t 600
```

##e4defrag
```shell
#For ext4 file systems
#Defag a single file part of a ext4 file system
e4defrag -v largfile

#Defrag against ext4 file system
e4defrag -v /path/to/mount/point
e4defrag -v /dev/vdb1

#defrag a directory in a ext4 file system
e4defrag -v /home/user/tmp

#Calculate defragmentation to determine if necessary
e4degrag -c /mnt/ext4
```

##mkfs/mount
```shell
#The logdev mount option is used to specify an external journal.
mkfs -t xfs -l logdev=/dev/sdd1 /dev/sdc1

#Mount a ext4 file system with an external journal
mount -o logdev=/dev/sdd1 /dev/sdc1 /mnt
```

##qperf
```shell
#The listener for qperf
qperf

#The sender for qperf
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