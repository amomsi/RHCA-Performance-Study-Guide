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
Important subdirectories in /proc/sys -- on pg.67
*Always use echo to add to these files*
For example:
echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_all
cat /proc/sys/net/ipv4/tcp_[rw]mem -- these are the start,min,max.
Could change that using this command:
echo "8192 87380 6291456" > /proc/sys/net/ipv4/tcp_rmem


###Manage /proc/sys tunables with sysctl
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
modprobe usb_storage dealy_use=5
This is a good way to test, we still need to make it perminent.
vi /etc/modprobe.d/usb_storage.conf
----usb_storage.conf
options usb_storage dealy_use=4
----
modprobe usb_storage


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
just make a directory there, and make tuned.conf

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






