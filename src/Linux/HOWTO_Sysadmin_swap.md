# 1. Linux HOWTO - SWAP Memory Manipulation

- [1. Linux HOWTO - SWAP Memory Manipulation](#1-linux-howto---swap-memory-manipulation)
  - [1.1. Introduction](#11-introduction)
  - [1.2. Kernel](#12-kernel)
  - [1.3. SWAP Memory](#13-swap-memory)
  - [1.4. Digging deeper](#14-digging-deeper)
  - [1.5. Conclusion](#15-conclusion)
  - [1.6. Copyright (c)](#16-copyright-c)

## 1.1. Introduction

Swap exists to allow for systems to have a bit of a "buffer" for if it runs out of Physical RAM. It ised to be much more important back when we had fairly large drive but small DRAM, where the RAM to Disk rtio was oin the 1000 or even 1000x to 1. OS,s will then try to push some of its static portions of the Virtal Memory into disk, to reduce the pressure on mdemory. The downside, of course is speed, where the dik is 1000x of times slower than Physical ram. Back thne, you owuld have 1GB of ram, and a 8GB SWAP size.

However in more conteporary systems, where RAM prices have dropped drastically, it isn't uncommong to see server with 256GB of ram (hell, even my laptop has 64 and my Desktop has 128GB of DDR4), all of the while, the disk size has not changed very drastically. Coupled with the slow nature of drives (and while SSDs might be quick, the non-NVME interface is still 6 to 12gbps wide only), making swap kind of a moot point in a lot of scenarios.

Virtual Machines, "Pods" and the impact of SSD is Not in the scope of this document.

## 1.2. Kernel  

The Kernel will control how, when and why memory gets swapped. A major way to control this behaviour is by adjusting the "vm.swappiness" sysctl knob. This setting tells the kernel how eager it should be in rying to push Virtual Memory[1] into disk, and it is a variable that can range from 0 (do not swap) to 100 (swap as freely as you wish). Each distribution will be different, but as an example, Ubuntu's default its memory swapping eagerness to 60, while netsapiens running servers will have this setting defaulted to 10.

`You can check what your system is set to, which would cause the following output. In this case the server is set to a eagerness to swap of 10%.`

```bash
$ sudo sysctl vm.swappiness
vm.swappiness = 10
```

`Changing this setting is very easly done using the same tool. So, in our case, if we wanted to set it to 0, we would do with the following output.`

```bash
$ sudo sysctl vm.swappiness=0
vm.swappiness = 0
```

It is important to know that sysctl can be manipulaed both live via the console, and by controlling scripts. if you are going to make a change to this setting, I recommend checking you /etc/sysctl.conf and all files inside the /etc/sysctl.d/ folder, which could be used to override manually set variables.

## 1.3. SWAP Memory

A system can have more than one swap memory mapped into the kernel, and the kernel will automatically distribute around those files.

`To get the Server's global memory status:`

```bash
$ free -h
              total        used        free      shared  buff/cache   available
Mem:           188G        121G        7.9G        320M         59G         66G
Swap:           63G        2.8G         61G
```

You can disable and enable swap on the fly, and as long as you have enough RAM to load the objects back into the physical memory, there is very little impact to doing it to a running system.

`To disable all configured SWAP memory:`

```bash
$ sudo swapoff -a && echo "OK" || echo "ERROR"
OK
```

`Enable all configured SWAP:`

```bash
$ sudo swapon -a && echo "OK" || echo "ERROR"
OK
```

## 1.4. Digging deeper

`A one linner that will allow you do check which processes, if anym are occupying SWAP space, order by the most to least used, and limit the display to the top 20 contenders:`

```bash
$ for file in /proc/*/status; do awk '/VmSwap|Name/{printf $2 " " $3}END{ print ""}' $file 2>/dev/null; done | sort -k 2 -n -r | head -10
mysqld 992056 kB
dockerd 25704 kB
memcached 20504 kB
nsnode 18852 kB
containerd 15708 kB
collectd 13640 kB
apache2 9520 kB
snapd 8440 kB
apache2 8220 kB
apache2 8200 kB
```

In this case, we can see mysql, docket and memcached being the highest on this list...

## 1.5. Conclusion

Swap memory can be a lifesaver for smaller and/or older systems, but in the current BigIron enviroment, it's performance compromise can more of a detriment than any usefulness than it can provide.

## 1.6. Copyright (c)

Copyright (c) 2022 P2P Tech, LLC.
All Rights reserved.

---

[1]: Do not confuse Virtual Memory with Virtual Machine. Morden OS's will MAP its physical RAM into multiple Virtual Memory sets for a few reasons, including System Stability, Security, Hardware Abstraction. [https://en.wikipedia.org/wiki/Virtual_memory]

<!-- 
```metadata
Creator: Thiago Modelli <thiago@modelli.us>
Created: May 12, 2022
Product: Linux
Type: HOWTO
Tags: [ KB, HOWTO, Linux, Swap, Performance, Tunning, SysAdm ]
Dificulty: Easy
Backgroup: Basic Linux
Contributions:
- [[COMMIT]] [[DATE]] [[FULLNAME]] [[EMAIL]]
``` -->
