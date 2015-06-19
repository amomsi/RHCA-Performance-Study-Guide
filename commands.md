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

#systemctl
```shell
systemctl set-property httpd.service MemoryLimit=512M
systemctl daemon-reload
```

#valgrind
```shell
yum install valgrind
valgrind --tool=cachegrind <command>
```

#systemtap
```shell
yum install systemtap kernel-devel
debuginfo-install kernel
```

#ps
```shell
#Show the virtual and physical allocation of a specific process.
ps up <pid>

#Can be used to view major and minor page faults
ps o pid,comm,minflt,majflt <pid>



```

#x86info
```shell
#Can be used to determine the size of the TLB buffer
x86info -c
```


