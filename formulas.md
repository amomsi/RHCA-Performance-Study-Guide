#Formulas


##Calculate bandwidth delay product
```shell
NIC speed * round-trip-time * 1/8 Byte/bits = BDP
```

##Swap Tendency
```shell
swap_tendency = mapped_ratio/2 + distress + vm_swappiness
```

##Balancing Interrupts
```shell
printf '%0x' $[2**0 + 2**2 + 2**7] > /proc /irq/50/smp_affinity
```

##Stripe Unit and Width
```shell
#Stride is the number of file system blocks that fit inside one chunk. An an example, for a file system block size of 4KiB and a chunk size of 64 KiB, the stride will be:
stride = 64KiB/4KiB = 16 blocks

#Stripe width is the number of file system blocks tha tfit on one stripe of the RAID array. For example, imagine a six disk RAID 6 array. By definition, in each stripe of RAID 6 array, two disks contain parity data. For the stripe-width, it is necessary to know how many disks in each stripe actually carry data blocks, so that will be 6 disks - 2 parity disks = 4 data disks. Each of those four disk will have ```stride``` number of file system blocks per chunk, so calculate 4(disks) x 16(stride) = 64 file system blocks per stripe.
stride width = 4(disks) x 16(stride) = 64
```

##Bandwidth Delay Product
```shell
BDP =  <NIC speed> * <RTT(from ping)
#example
BDP = 1 Gb/s * 0.341ms =
10^9 b/s * 0.000341s =
341000 bits x 1/8 B/bits =
42625 Bytes = 41.63 KiB
```