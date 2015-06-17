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























