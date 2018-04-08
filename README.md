# Hacking Huawei HG8012H ONT
## Steps to hack a HG8012H, access it and mod the firmware


Due to the change from Vodafone to Masmóvil network that took place last year by Pepephone ISP, it was necessary to me to obtain an ADSL router compatible with the 802.1q protocol to be able to use the internet connection. This caused me to retire my Asus DSL-N55u router, with which I was very happy with its stability, configuration and WIFI coverage. So, I purchased a Huawei HG556a ADSL router and since this router had a 10/100 switch and pretty mediocre WIFI speed, I also purchased a Linksys EA8500 router that would provide me with the WIFI coverage and gigabit connectivity for my devices. So I configured the Huawei HG556a router in WAN Bridge mode and connected it to the WAN port of the Linksys router. This configuration gave me excellent performance, even more after flashing DD-WRT on the Linksys router.


Now, I have changed the internet operator, moving to a FTTH connection of 500/300 MB with Cableworld and they installed a Huawei HG8245U router and an optical splitter whose coaxial output, connected to home's TV splitter, distributes the TV channels of the service. As I was very happy with the operation and WIFI coverage of the Linksys Ea8500 router, I configured the HG8245U router again in WAN bridge mode and this is the way in which I have been working.

![GitHub Logo](https://github.com/logon84/Hacking_Huawei_HG8012H_ONT/blob/master/pics/1spliiter.jpg)

At this point several problems arise: The cableworld HUWAEI router is huge and I am lacking some space in the living room furniture, the use of two devices (HUAWEI router and optical splitter) forces me to use two power plugs, which increases the entanglement of cables in general. In addition, this router has a high power consumption, which is a waste of resources, since I am not using any of its ethernet outputs (except 1 connected to the Linksys WAN router), nor its Wi-Fi, nor 90% of its functions. So I started looking for a optical router/ONT with the slogan: small size, integrated CATV output, low power consumption and Huawei brand to be compatible with Cableworld's OLT. The perfect candidate was the Huawei HG8012H and in one of my trips to Portugal I was lucky enough to find one for € 5 in Cashconverters (normal price is about € 80).


![GitHub Logo](https://github.com/logon84/Hacking_Huawei_HG8012H_ONT/blob/master/pics/2HG8012h.jpg)


### Hostility level 0

I got down to work configuring this ONT with my ISP parameters, so I connected an ethernet cable between the ONT and my PC and tried to open http://192.168.1.1 in the browser, but I got no response. Then I searched the internet for Huawei documents and technical files and found that for this model, the default IP address is 192.168.100.1 and the access users are telecomadmin:admintelecom and root:admin. After making the relevant IP and subnet changes in the network card of my PC, I went to http://192.168.100.1 in the browser and voila, the WebUi appeared to be able to start configuring my brand new ONT:

![GitHub Logo](https://github.com/logon84/Hacking_Huawei_HG8012H_ONT/blob/master/pics/3login.png)

Unfortunately, none of the two access users I had found in the documentation worked. So it occurred to me to press the reset button for 30 seconds with a clip with the total certainty that this would cause the default access users to be functional again. But I was wrong. After a long time trying to search the internet for a login that would allow me access to the ONT, not only did I not find it but I verified that there is very little information about this device available. As if that were not enough, I discovered that when this ONT comes directly from an ISP, it is usually blocked by them to not allow access to the configuration and that way the user can not reuse it with another ISP. In this case, it appeared that the ONT that I bought in Cashconverters had not been purchased directly from HUAWEI but had been installed by a supplier. Things started to get complicated.


### Hostility level 1

I could see that by entering 3 incorrect passwords, the router did not allow retry the login until 1 minute after, so the option of dictionaries and brute force attacks to access was discarded by the amount of time the entire process would take.

Normally many routers allow telnet access to the device as an alternative way of configuration, but in this case it was impossible and the connection closed due to lack of response from the ONT.

Trying to find some weak point in the ONT that allowed me access, I did a port scanner from my PC, with the following command:

nmap -Pn -n -p0- 192.168.100.1


It did not help much, all ports were closed except port 80 (webUi). Port 23 (telnet) was not only not open but was being filtered by the integrated firewall to further complicate things. There was nothing else I could do externally to solve this, so screwdriver in hand I ventured to examine the bowels of the bug.

## Hostility level 2

Once the cover was opened and after a component identification phase, this is what I found:

![GitHub Logo](https://github.com/logon84/Hacking_Huawei_HG8012H_ONT/blob/master/pics/4bottom_PCB.jpg)


![GitHub Logo](https://github.com/logon84/Hacking_Huawei_HG8012H_ONT/blob/master/pics/5spec.jpg)

Of all these components the ones that caught my attention were the serial port pads, the JTAG port and the flash memory. After a search on the internet, I deduced the pinout of both the serial port and the JTAG. They are quite common among HUAWEI router devices.

![GitHub Logo](https://github.com/logon84/Hacking_Huawei_HG8012H_ONT/blob/master/pics/6jtag.png)

![GitHub Logo](https://github.com/logon84/Hacking_Huawei_HG8012H_ONT/blob/master/pics/7serial.jpg)

I tested JTAG portusing the parallel port cable that can be seen in the previous photo but I did not get response, so I focused my efforts on the serial port. After connecting a TTL-USB converter and connecting to the virtual COM port using PutTy with a rate of 115200 symbols, I started to see the following bootlog on the screen:

>HuaWei StartCode 2012.02 (Mar 25 2014 - 01:04:34)
>
>SPI:
>startcode select the uboot to load
>the high RAM is :8080103c
>startcode uboot boot count:0
>Boot load address :0x80000
>Use the UbootB to load success
>
>
>U-Boot 2010.03 (R13C10 Jul 31 2015 - 14:38:54)
>
>DRAM:  128 MB
>Boot From SPI flash
>Chip Type is SD5115S
>SFC : cs0 unrecognized JEDEC id 00ffffff, extended id 00000000
>SFC: extend id 0x300
>SFC: cs1 s25sl12800 (16384 Kbytes)
>SFC: Detected s25sl12800 with page size 262144, total 16777216 bytes
>SFC: already protect ON !
>SFC: sfc_read flash offset 0x80000, len 0x40000, memory buf 0x81fa0008
>*** Warning - bad CRC, using default environment
>
>In:    serial
>Out:   serial
>Err:   serial
>PHY power down !!!
>[main.c__5566]::CRC:0x6a8fe445, Magic1:0x5a5a5a5a, Magic2:0xa5a5a5a5, count:0, CommitedArea:0x1, Active:0x1, RunFlag:0x0
>SFC : cs0 unrecognized JEDEC id 00ffffff, extended id 00000000
>SFC: extend id 0x300
>SFC: cs1 s25sl12800 (16384 Kbytes)
>SFC: Detected s25sl12800 with page size 262144, total 16777216 bytes
>initialize flash success
>Start from main system(0x1)!
>CRC:0x6a8fe445, Magic1:0x5a5a5a5a, Magic2:0xa5a5a5a5, count:1, CommitedArea:0x1, Active:0x1, RunFlag:0x0
>Main area (B) is OK!
>CRC:0xc4e775d4, Magic1:0x5a5a5a5a, Magic2:0xa5a5a5a5, count:1, CommitedArea:0x1, Active:0x1, RunFlag:0x0
>iRootfsSize to 0x47c1d4
>Start copy data from 0x1c9c0054 to 0x86000000 with sizeof 0x0047c1d4 ............Done!
>Bootcmd:bootm 0x1c340054 0x86000000
>BootArgs:noalign mem=114M console=ttyAMA1,115200 initrd=0x86000040,0x47c194 rdinit=/linuxrc mtdparts=hi_sfc:0x40000(startcode),0x40000(bootA)ro,0x40000(bootB)ro,0x40000(flashcfg)ro,0x40000(slave_param)ro,0x200000(kernelA)ro,0x200000(kernelB)ro,0x480000(rootfsA)ro,0x480000(rootfsB)ro,0x180000(file_system),-(reserved)pcie1_sel=x1 maxcpus=0 user_debug=0x1f panic=1
>U-boot Start from NORMAL Mode!
>
>## Booting kernel from Legacy Image at 1c340054 ...
>   Image Name:   Linux-2.6.34.10_sd5115v100_wr4.3
>   Image Type:   ARM Linux Kernel Image (uncompressed)
>   Data Size:    2025204 Bytes =  1.9 MB
>   Load Address: 81000000
>   Entry Point:  81000000
>## Loading init Ramdisk from Legacy Image at 86000000 ...
>   Image Name:   cpio
>   Image Type:   ARM Linux RAMDisk Image (uncompressed)
>   Data Size:    4702612 Bytes =  4.5 MB
>   Load Address: 00000000
>   Entry Point:  00000000
>SFC : cs0 unrecognized JEDEC id 00ffffff, extended id 00000000
>SFC: extend id 0x300
>SFC: cs1 s25sl12800 (16384 Kbytes)
>SFC: Detected s25sl12800 with page size 262144, total 16777216 bytes
>   Loading Kernel Image ... SFC: sfc_read flash offset 0x340094, len 0x1ee6f4, memory buf 0x81000000
>OK
>OK
>
>Starting kernel ...
>
>Uncompressing Linux... done, booting the kernel.
>Kernel Early-Debug on Level 0
> V: 0xF1100000 P: 0x00010100 S: 0x00001000 T: 0
> V: 0xF110E000 P: 0x0001010E S: 0x00001000 T: 0
> V: 0xF110F000 P: 0x0001010F S: 0x00001000 T: 0
> V: 0xF1104000 P: 0x00010104 S: 0x00001000 T: 0
> V: 0xF1180000 P: 0x00010180 S: 0x00002000 T: 0
> V: 0xF1400000 P: 0x00010400 S: 0x00001000 T: 12
>early_init      72      [arch/arm/mach-sd5115h-v100f/core.c]
>sd5115_map_io   223     [arch/arm/mach-sd5115h-v100f/core.c]
>smp_init_cpus   163     [arch/arm/mach-sd5115h-v100f/platsmp.c]
>sd5115_gic_init_irq     88      [arch/arm/mach-sd5115h-v100f/core.c]
>sd5115_timer_init       471     [arch/arm/mach-sd5115h-v100f/core.c]
>sd5115_clocksource_init 451     [arch/arm/mach-sd5115h-v100f/core.c]
>twd_base :
>0xF1180600
>sd5115_timer_init       491     [arch/arm/mach-sd5115h-v100f/core.c]
>smp_prepare_cpus        174     [arch/arm/mach-sd5115h-v100f/platsmp.c]
>hi_kernel_wdt_init      207     [arch/arm/mach-sd5115h-v100f/hi_drv_wdt.c]
>sd5115_init     314     [arch/arm/mach-sd5115h-v100f/core.c]
>sd5115_init     320     [arch/arm/mach-sd5115h-v100f/core.c]
>sd5115_init     320     [arch/arm/mach-sd5115h-v100f/core.c]
>sd5115_init     327     [arch/arm/mach-sd5115h-v100f/core.c]
>sd5115_init     330     [arch/arm/mach-sd5115h-v100f/core.c]
>Linux version 2.6.34.10_sd5115v100_wr4.3 (root@wuhci2lslx00096) (gcc version 4.4.6 (GCC) ) #1 SMP Fri Jul 31 14:38:51 CST 2015
>CPU: ARMv7 Processor [413fc090] revision 0 (ARMv7), cr=10c53c7f
>CPU: VIPT nonaliasing data cache, VIPT nonaliasing instruction cache
>Machine: sd5115
>Memory policy: ECC disabled, Data cache writealloc
>sd5115 apb bus clk is 100000000
>PERCPU: Embedded 7 pages/cpu @c04d9000 s4448 r8192 d16032 u65536
>pcpu-alloc: s4448 r8192 d16032 u65536 alloc=16*4096
>pcpu-alloc: [0] 0
>Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 28956
>Kernel command line: noalign mem=114M console=ttyAMA1,115200 initrd=0x86000040,0x47c194 rdinit=/linuxrc mtdparts=hi_sfc:0x40000(startcode),0x40000(bootA)ro,0x40000(bootB)ro,0x40000(flashcfg)ro,0x40000(slave_param)ro,0x200000(kernelA)ro,0x200000(kernelB)ro,0x480000(rootfsA)ro,0x480000(rootfsB)ro,0x180000(file_system),-(reserved)pcie1_sel=x1 maxcpus=0 user_debug=0x1f panic=1
>PID hash table entries: 512 (order: -1, 2048 bytes)
>Dentry cache hash table entries: 16384 (order: 4, 65536 bytes)
>Inode-cache hash table entries: 8192 (order: 3, 32768 bytes)
>Memory: 114MB = 114MB total
>Memory: 107068k/107068k available, 9668k reserved, 0K highmem
>Virtual kernel memory layout:
>    vector  : 0xffff0000 - 0xffff1000   (   4 kB)
>    fixmap  : 0xfff00000 - 0xfffe0000   ( 896 kB)
>    DMA     : 0xffc00000 - 0xffe00000   (   2 MB)
>    vmalloc : 0xc7800000 - 0xd0000000   ( 136 MB)
>    lowmem  : 0xc0000000 - 0xc7200000   ( 114 MB)
>    modules : 0xbf000000 - 0xc0000000   (  16 MB)
>      .init : 0xc0008000 - 0xc002b000   ( 140 kB)
>      .text : 0xc002b000 - 0xc0396000   (3500 kB)
>      .data : 0xc03aa000 - 0xc03c6660   ( 114 kB)
>SLUB: Genslabs=11, HWalign=32, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
>Hierarchical RCU implementation.
>RCU-based detection of stalled CPUs is enabled.
>NR_IRQS:160
>Calibrating delay loop... 747.11 BogoMIPS (lpj=3735552)
>Security Framework initialized
>Mount-cache hash table entries: 512
>CPU: Testing write buffer coherency: ok
>Init trace_clock_cyc2ns: precalc_mult = 312500, precalc_shift = 8
>Brought up 1 CPUs
>SMP: Total of 1 processors activated (747.11 BogoMIPS).
>hi_wdt: User-Mode!
>hi_wdt: Init sucessfull!
>NET: Registered protocol family 16
>id:0x51151100
>check_res_of_trace_clock: sched_clock() high resolution
>Serial: dw  uart driver
>uart:0: ttyAMA0 at MMIO 0x1010e000 (irq = 77) is a AMBA/DW
>uart:1: ttyAMA1 at MMIO 0x1010f000 (irq = 78) is a AMBA/DW
>console [ttyAMA1] enabled
>bio: create slab <bio-0> at 0
>vgaarb: loaded
>usbcore: registered new interface driver usbfs
>usbcore: registered new interface driver hub
>usbcore: registered new device driver usb
>cfg80211: Calling CRDA to update world regulatory domain
>Switching to clocksource timer1
>NET: Registered protocol family 2
>IP route cache hash table entries: 128 (order: -3, 512 bytes)
>TCP established hash table entries: 4096 (order: 3, 32768 bytes)
>TCP bind hash table entries: 4096 (order: 3, 32768 bytes)
>TCP: Hash tables configured (established 4096 bind 4096)
>TCP reno registered
>UDP hash table entries: 128 (order: 0, 4096 bytes)
>UDP-Lite hash table entries: 128 (order: 0, 4096 bytes)
>NET: Registered protocol family 1
>RPC: Registered udp transport module.
>RPC: Registered tcp transport module.
>RPC: Registered tcp NFSv4.1 backchannel transport module.
>Trying to unpack rootfs image as initramfs...
>Freeing initrd memory: 4592K
>squashfs: version 4.0 (2009/01/31) Phillip Lougher
>JFFS2 version 2.2. © 2001-2006 Red Hat, Inc.
>msgmni has been set to 218
>io scheduler noop registered
>io scheduler deadline registered
>io scheduler cfq registered (default)
>brd: module loaded
>mtdoops: mtd device (mtddev=name/number) must be supplied
>Spi id table Version 1.22
>Spi Flash Controller V300 Device Driver, Version 1.10
>Spi(cs1) ID: 0x01 0x20 0x18 0x03 0x00 0x00
>Spi(cs1): Block:256KB Chip:16MB (Name:S25FL128P-0)
>Lock Spi Flash(cs1)!
>Hisilicon flash: registering whole flash at once as master MTD
>mtd: bad character after partition (p)
>11 cmdlinepart partitions found on MTD device hi_sfc
>Creating 11 MTD partitions on "hi_sfc":
>0x000000000000-0x000000040000 : "startcode"
>0x000000040000-0x000000080000 : "bootA"
>0x000000080000-0x0000000c0000 : "bootB"
>0x0000000c0000-0x000000100000 : "flashcfg"
>0x000000100000-0x000000140000 : "slave_param"
>0x000000140000-0x000000340000 : "kernelA"
>0x000000340000-0x000000540000 : "kernelB"
>0x000000540000-0x0000009c0000 : "rootfsA"
>0x0000009c0000-0x000000e40000 : "rootfsB"
>0x000000e40000-0x000000fc0000 : "file_system"
>0x000000fc0000-0x000001000000 : "reserved"
>Special nand id table Version 1.33
>Hisilicon Nand Flash Controller V301 Device Driver, Version 1.10
>PPP generic driver version 2.4.2
>PPP Deflate Compression module registered
>PPP BSD Compression module registered
>PPP MPPE Compression module registered
>NET: Registered protocol family 24
>SLIP: version 0.8.4-NET3.019-NEWTTY (dynamic channels, max=256) (6 bit encapsulation enabled).
>CSLIP: code copyright 1989 Regents of the University of California.
>SLIP linefill/keepalive option.
>Netfilter messages via NETLINK v0.30.
>ip_tables: (C) 2000-2006 Netfilter Core Team
>arp_tables: (C) 2002 David S. Miller
>TCP cubic registered
>NET: Registered protocol family 17
>Freeing init memory: 140K
>
>                        -=#  DOPRA LINUX 1.0  #=-
>                        -=#  EchoLife WAP 0.1  #=-
>                        -=#  Huawei Technologies Co., Ltd #=-
>
>mount file system
>Loading the kernel modules:
>Loading module: rng-core
>modprobe: chdir(2.6.34.10_sd5115v100_wr4.3): No such file or directory
>Loading module: nf_conntrack
>modprobe: chdir(2.6.34.10_sd5115v100_wr4.3): No such file or directory
>Loading module: xt_mark
>modprobe: chdir(2.6.34.10_sd5115v100_wr4.3): No such file or directory
>Loading module: xt_connmark
>modprobe: chdir(2.6.34.10_sd5115v100_wr4.3): No such file or directory
>Loading module: xt_MARK
>modprobe: chdir(2.6.34.10_sd5115v100_wr4.3): No such file or directory
>Loading module: xt_limit
>modprobe: chdir(2.6.34.10_sd5115v100_wr4.3): No such file or directory
>Loading module: xt_state
>modprobe: chdir(2.6.34.10_sd5115v100_wr4.3): No such file or directory
>Loading module: xt_tcpmss
>modprobe: chdir(2.6.34.10_sd5115v100_wr4.3): No such file or directory
>Loading module: nf_nat
>modprobe: chdir(2.6.34.10_sd5115v100_wr4.3): No such file or directory
>Loading module: ipt_MASQUERADE
>modprobe: chdir(2.6.34.10_sd5115v100_wr4.3): No such file or directory
>Loading module: ipt_REDIRECT
>modprobe: chdir(2.6.34.10_sd5115v100_wr4.3): No such file or directory
>Loading module: ipt_NETMAP
>modprobe: chdir(2.6.34.10_sd5115v100_wr4.3): No such file or directory
>Loading module: nf_conntrack_ipv4.ko
>modprobe: chdir(2.6.34.10_sd5115v100_wr4.3): No such file or directory
>Loading module: iptable_nat.ko
>modprobe: chdir(2.6.34.10_sd5115v100_wr4.3): No such file or directory
>Making device instances:
>Setting console log message level:
>Setting hostname:
>Settingup the sysctl configurations:
>Setting up interface lo:
>Running local startup scripts.
>
>*******************************************
>--==        Welcome To IAS WAP         ==--
>--==   Huawei Technologies Co., Ltd.   ==--
>*******************************************
>IAS WAP Ver:V800R013C10SPC182B001
>IAS WAP Timestamp:2015/07/30 00:08:53
>*******************************************
>
>Start init IAS WAP basic module ....
>current lastword info:Add=0xc7a06000;max_num=300;Add1=0xc7a01000;Add2=0xc7a06000;Add3=0xc7a0b000;
>Init IAS WAP basic module done!
>soft lockup args:snap=150; release=50; dump flag=1;
>Set kmsgread process pid to:92;
>UBIFS error (pid 102): ubifs_get_sb: cannot open "/dev/ubi0_13", error -22
>mount: mounting /dev/ubi0_13 on /mnt/jffs2/ failed: Invalid argument
>umount /mnt/jffs2
>umount: can't umount /mnt/jffs2: Invalid argument
>UBIFS error (pid 105): ubifs_get_sb: cannot open "/dev/ubi0_13", error -22
>mount: mounting /dev/ubi0_13 on /mnt/jffs2/ failed: Invalid argument
>Mount nor jffs2 in 1.sdk_init...
>fenghe.linux4.3
>Get kernel version:2.6.34
>Rootfs time stamp:2015-07-31_14:40:38
>SVN label(ont):/etc/rc.d/rc.start/1.sdk_init.sh: line 50: can't create /proc/sys/vm/pagecache_ratio: nonexistent directory
>User init start......
>Loading the SD5115V100 modules:
>
> SYSCTL module is installed
>
> PIE module is installed
>
> GPIO module is installed
>
> SPI module is installed
>
> I2C module is installed
>
> DP module is installed
>
> MDIO module is installed
>
> TIMER module is installed
>
> UART module is installed
>
> HW module is installed
>ifconfig eth0 hw ether 18:C5:8A:A0:28:4C
>Loading the EchoLife WAP modules: LDSP
>COMMON For LDSP Install Successfully...
>cut kernel config
>major-minor:10-58
>mknod: /dev/hlp: File exists
>GPIO For LDSP Install Successfully...
>sh: 0: unknown operand
>
> ------ SOC is 5115 S PILOT ------
><ldsp>board version is 5
><ldsp>pcb version is 0
><ldsp>orig board version is 5
>CHIPADP-SD5115 BASIC For LDSP Install Successfully...
>CHIPADP-SD5115 EXT For LDSP Install Successfully...
>I2C For LDSP Install Successfully...
>LSW L2 For LDSP Install Successfully...
>LSW L3 For LDSP Install Successfully...
>DEV For LDSP Install Successfully...
>[DM]:ae_chip[0]=4,ae_chip[1]=255,ae_chip[2]=255,ae_chip[3]=0
>[DM]:board_ver=5,pcb_ver=0
>hw_dm_init_data successfully...
>
> [ /mnt/jffs2/boardinfocustom.cfg not exsit ! not need deal.]
>hw_dm_pdt_init successfully...
>hw_feature_init begin...
>hw_feature_proc_init begin...
>hw_feature_data_init begin...
>ac_cfgpath is not null,acTmpBuf=/etc/wap/customize/ptvdfb_ft.cfg.....!
>ac_hard_cfgpath is not null, acTmpBuf=/mnt/jffs2/hw_hardinfo_feature.bak.....!
>ac_cfgpath is not null,acTmpBuf=/etc/wap/customize/spec_ptvdfb.cfg.....!
>ac_hard_cfgpath is not null, acTmpBuf=/mnt/jffs2/hw_hardinfo_spec.bak.....!
>hw_feature_init Successfully...
>pots_num=0
>ssid_num=0
> usb_num=0
>hw_route=0
>   l3_ex=1
>    ipv6=0
>Read MemInfo Des: 1118
>SPI For LDSP Install Successfully...
>UART For LDSP Install Successfully...
>BATTERY For LDSP Install Successfully...
>OPTIC For LDSP Install Successfully...
>PLOAM For LDSP Install Successfully...
>GMAC For LDSP Install Successfully...
>KEY For LDSP Install Successfully...
>LED For LDSP Install Successfully...
>RF For LDSP Install Successfully...
>Loading BBSP L2 modules:
>PTP For BBSP Install Successfully...
>hw_igmp_kernel Install Successfully...
>
> dhcp_module_init load success !
>
> pppoe_module_init load success !
>hw_ringchk_kernel Install Successfully...
>hw_portchk_kernel Install Successfully...
>l2base For BBSP Install Successfully...
>vbr_unicast_car:50
>vbr_unicast_car:50
>vbr_unicast_car:50
>Pktdump init Install Successfully...
>
> hw_cpu_usage_install
>[ker_L2M_CTP] for bbsp Install Successfully...
>EMAC For LDSP Install Successfully...
>MPCP For LDSP Install Successfully...
>Loading BBSP L2_extended modules:
>hw_ethoam_kernel Install Successfully...
>l2ext For BBSP Install Successfully...
>Dosflt For BBSP Install Successfully...
>
> vlanflt_module_init load success !
>l3base for bbsp Install Successfully...
>1.sdk_init.sh close core dump, flag=
>Start ldsp_user...0
><LDSP> system has no slave space for bob
><LDSP_CFG> Set uiUpMode=1 [1:GPON,2:EPON,4:AUTO]
>SD511X test self OK
>Extern Lsw test self NoCheck
>Optic test self OK
>WIFI test self NoCheck
>PHY[1] test self OK
>PHY[2] test self OK
>PHY[3] test self OK
>PHY[4] test self OK
>PHY[5] test self NoCheck
>PHY[6] test self NoCheck
>
>  LINE = 204, FUNC = hi_kernel_i2c_burst_read_bytes
> read data is over time
><LDSP> common optic,the last i2c error is normal,donot worry
>
><LDSP> uiRet = 2 pcNodeName = Cfg1 Cmd = 20005000 Length = 10 Value = be8406b2
> GPON init success !
>ssmp bbsp igmp amp ethoam omci
>Start start pid=252; uiProcNum=6;
>InitFrame omci; PID=256; state=0; 15.084;
>InitFrame omci; PID=256; in state=0; 15.084;
>InitFrame omci; PID=256; out state=0; 15.085;
>InitFrame ssmp; PID=253; state=0; 15.172;
>InitFrame ssmp; PID=253; in state=0; 15.173;
>uiCfgAddr:c0000
><db/hw_xml_dbmain.c:7713>acChooseWord:NOCHOOSE UserChoiceFlag:-1 Updateflag:-1
>
>InitFrame bbsp; PID=254; state=0; 15.744;
>InitFrame bbsp; PID=254; in state=0; 15.745;
>InitFrame bbsp; PID=254; out state=0; 15.747;
>InitFrame ethoam; PID=258; state=0; 15.750;
>InitFrame ethoam; PID=258; in state=0; 15.750;
>InitFrame ethoam; PID=258; out state=0; 15.752;
>InitFrame igmp; PID=257; state=0; 15.759;
>InitFrame igmp; PID=257; in state=0; 15.759;
>InitFrame igmp; PID=257; out state=0; 15.759;
>InitFrame amp; PID=255; state=0; 15.974;
>InitFrame amp; PID=255; in state=0; 15.975;
>InitFrame amp; PID=255; out state=0; 15.976;
><db/hw_xml_dbmain.c:8784>acFilePath:/etc/wap/hw_aes_tree.xml pstRoot:0x0
><db/hw_xml_dbmain.c:9068>acFilePath:/etc/wap/hw_aes_tree.xml pstRoot:0x3835ebc4
>pfFuncHandle ERR. uiRet:ffffffff;
> pfFuncHandle ERR. uiRet:ffffffff;
> pfFuncHandle ERR. uiRet:ffffffff;
> pfFuncHandle ERR. uiRet:ffffffff;
> pfFuncHandle ERR. uiRet:ffffffff;
> pfFuncHandle ERR. uiRet:ffffffff;
> pfFuncHandle ERR. uiRet:ffffffff;
> pfFuncHandle ERR. uiRet:ffffffff;
> pfFuncHandle ERR. uiRet:ffffffff;
> pfFuncHandle ERR. uiRet:ffffffff;
> pfFuncHandle ERR. uiRet:ffffffff;
> <db/hw_xml_dbmain.c:7100>[HW_XML_DBOnceSave] Set DB Auto Save in 12000 ticks.
>Reset reason: unknown reason, except oom, watchdog and lossing power!
