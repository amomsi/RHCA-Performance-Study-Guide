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