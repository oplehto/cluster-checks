# cluster-checks
A collection of checks for cluster homogenity. I'll do some fancy checks at some point but right now it's in a checklist form with some examples (right now only for HP clusters).

## Background
In order to ensure the hassle-free of a cluster, it's important that the baseline system is in a very consistent shape. This means that the nodes are identical as possible. Oftentimes there are some inconsistencies which may have been missed in the factory integration tests. Sometimes the issues may not cause imminent errors but may cause trouble down the line. Thus it will be very important to weed these out as soon as possible. During my years as managing HPC clusters I've become quite good at catching these and thought it would be good to share the knowledge. 

Sites buying clusters should always demand in their contracts that cluster nodes must have consistent configuration and have a checklist to verify this. However, testing should not be only limited to prescribed checklist items as new sources of inconsistency may be discovered. Use your creativity. :)

I also would be very happy if any vendor reading this would pass it on to their test teams. 
## Test checklist 
The checklist is divided into categories and examples given. They are currently very HP-specific but there should be similar commands on other vendors' hardware. Any feedback and improvements are very welcome. 

The example commands assume that you have pdsh installed (https://computing.llnl.gov/linux/pdsh.html) and the cluster hosts defined in the pdsh host list. 

It is also possible to use dshbak / pshbak / dbuck to aggregate pdsh output (latter two from https://www.nsc.liu.se/~kent/python-hostlist/ )

The scripts assume that pdsh is used to gather the information. The commands typically provide fairly large dumps of data. It's left to the task of the reader to pipe the output to a sort command or do diffs to evaluate the consistency. In many cases a simple `sort`, `grep` or `wc -l` is already useful but, for example in parsing `dmidecode` and `dmesg` output doing `diff`s might be better. I'll try to get around to productizing this at some point as well.

### PDU
Even PDUs have firmware these days and this gets often overlooked updates or when replacing hardware. This may cause also weird discrepancies in power consumption or differences in the sensor reports.
- Number of operational PDUs and their status
 - ```pdsh -a 'hplog -p'```
- PDU power draw 
- PDU firmware
 - ```pdsh -a '/shared/HP/ppic -d | grep Firmware'```
 - ```pdsh -a '/shared/HP/ppic -d | grep Bootblock'```

### Fans
Fans are one of the most commonly failing component. Sometimes a partial failure may also be indicated with a strange state or a RPM reading. 
- Operational fans per node
 - ```pdsh -a 'hplog -f '```
- RPMs of fans

### Temperature sensors
Temperature sensors can be miscalibrated or malfunctioning (completely blank). Sometimes the non-homogeneous output is indicative of a different firmware version. 
- Number of functional temperature sensors 
 - ```pdsh -a 'hplog -t | grep -- "---" | wc -l'```
- Temperatures should be consistent
 -  ```pdsh -a 'hplog -t' | sort -k 8 -n | tail -n 10'```

### Processors and system
Processors typically do not have major issues but it's good to run a proper burn-in test. These can also expose some other low-level issues which the other tests might not find. 
 - [HPL (High-Performance Linpack)](http://www.netlib.org/benchmark/hpl/) on each node
 - [NAS Parallel Benchmarks](http://www.nas.nasa.gov/publications/npb.html) 
 

### Disks
Disks may have performance problems as well as degradation. Luckily the disks have self-tests and counters to find these out. The counters also can expose if you are being sold a used disk as new :)
- Quick test of disk bandwidth
 - ```pdsh -a 'smartctl -d scsi --all /dev/sda|egrep "read:|write:"'```
- Elements in the grown defect list
 - ```pdsh -a 'smartctl -d scsi --all /dev/sda|egrep "Elements in grown defect list"' | sort -n -k 7```
- Elements in reallocated sector count
 - ```pdsh -a 'smartctl --attributes -d cciss,0 /dev/sda' | grep Reallocated_Sector_Ct | sort -n -k 11```
- Short self test 
 - ```pdsh -a 'smartctl -t short -d cciss,0 /dev/sda'```
- Geometry of disks
 - ```pdsh -a 'hdparm /dev/sda|grep geometry'```
- Operational hours and number of reboots
 - ```pdsh -a 'smartctl -a /dev/sda |grep Power_Cycle_Count'```
 - ```pdsh -a 'smartctl -a /dev/sda |grep Power_On_Hours'```

- Disk firmware version
- RAID controller firmware version 

### Management processor
The management processor logs are a useful place to discover pre-existing issues on the systems. It's also good to keep monitoring it during the initial operation of the system to see if there are any strange messages.
- Firmware version
 -  ```pdsh -a 'ipmitool mc info | grep Firmware'```
- Event log size
 - ```pdsh -a 'ipmitool sel list'```
- Unusual event log messages
 - ```pdsh -a 'ipmitool sel list'```
- Time correct
 - ```pdsh -a 'ipmitool sel time get'```
- Any new log messages in the first week of operation? 

### Memory
Differences in memory size are indicative of problems with DIMMs. In some cases even complete DIMMs may be missing or are the wrong size.

- Total amount of memory
 - ```pdsh -a 'cat /proc/meminfo | grep MemTotal'```
- Differences in DIMMs
 - ```dmidecode -t memory``` 
- [STREAM2 TRIAD](https://www.cs.virginia.edu/stream/stream2/) on each node 

### Ethernet
Network cards have their own firmware which often is not upgraded if the card is changed. Errors or wrong line rates can indicate failed cables or interfaces.

- Firmware version
 - ```pdsh -a 'ethtool -i eth0 | grep firmware'```
- Errors or drops
 - ```pdsh -a 'cat /sys/class/net/eth0/statistics/tx_dropped'``` 
 - ```pdsh -a 'cat /sys/class/net/eth0/statistics/rx_dropped'``` 
 - ```pdsh -a 'cat /sys/class/net/eth0/statistics/tx_errors'``` 
 - ```pdsh -a 'cat /sys/class/net/eth0/statistics/rx_errors'``` 
- Line rate
 - ```pdsh -a 'cat /sys/class/net/eth0/speed'``` 
- Firmware version of switches

### InfiniBand
Network cards have their own firmware which often is not upgraded if the card is changed. Errors or wrong line rates can indicate failed cables or interfaces. InfiniBand cables can be very sensitive and can break in a way that drops the line rate. 

- Firmware version of HCAs
 - ```pdsh -a 'ibstat | grep Firmware'```  
- Line rate
 - ```pdsh -a 'cat /sys/class/net/ib0/speed'``` 
- Errors or drops
 - ```pdsh -a 'cat /sys/class/net/ib0/statistics/tx_dropped'``` 
 - ```pdsh -a 'cat /sys/class/net/ib0/statistics/rx_dropped'``` 
 - ```pdsh -a 'cat /sys/class/net/ib0/statistics/tx_errors'``` 
 - ```pdsh -a 'cat /sys/class/net/ib0/statistics/rx_errors'``` 
- Firmware version of switches
- [mpisweep](https://github.com/CSC-IT-Center-for-Science/mpisweep) MPI connectivity benchmark 

### BIOS
Inconsistent BIOS settings and versions tend to be quite common. There are some differences in the lspci, lsusb and dmidecode output (for example serial numbers) but major discrepancies such as different line counts are indicative of possibly mainboard problems or missing components such as DIMMs (which should also show up in the memory size checks).

- BIOS version
 - ```pdsh -a 'dmesg | grep DMI:'```
- Output length of dmidecode
 - ```pdsh -a 'dmidecode | wc -l'```
- BIOS settings
 - HP/HPE servers: ```conrep``` or ```hprcu``` 
- PCI device report length
 - ```pdsh -a 'lspci -v |wc -l'``` 
- USB device report length
 - ```pdsh -a 'lsusb -v |wc -l'``` 

### GPUs
These are some things that can be checked on GPUs. Examples are NVidia specific.
- ECC Errors
 - ```nvidia-smi -a --xml-format | grep -A 33 "<ecc_errors>" | grep "<total>" | grep -v "<total>0</total>"````
- BIOS version
 - ```nvidia-smi -a --xml-format | grep vbios_version```
- Temperature sensors
 - ```nvidia-smi -a --xml-format | grep "<gpu_temp>"```
- ECC Mode 
 - ```nvidia-smi -a --xml-format | grep "<current_ecc>"``` 
- Power draw
 - ```nvidia-smi -a --xml-format | grep "<power_draw>"```
 
### Xeon Phi coprocessor card
A lot of the checks of the other sections can be applied to every Xeon Phi card as well because they are essentially diskless Linux boxes. In addition there are some host side checks. 
- The miccheck utility
 - ```miccheck````
- Coprocessor OS version
 - ```micsmc -i | grep "Coprocessor OS Version"``` 
- Flash version
 - ```micsmc -i | grep "Flash Version"````
- ECC status
 - ```micsmc --ecc````
- Total memory
 - ```micsmc -m | grep Total````
- Temperature sensors
 - ```micsmc -t````
- Power draw
 - ```micsmc -f | grep "Total Power"``` 

### OS -level
There are some things that should be checked on the OS side. Although the kernel version and rpm count does not indicate harware issues, such OS-level discrepancies may affect the reports provided by the other commands. Incorrect date may be an issue with the clock if NTP sync is enabled. 

- dmesg log length after a boot of the whole system and after idling the cluster for at least 3 hours
 - ```pdsh -a 'dmesg|wc -l' | sort -k 2 -n```
- Clocks synchronized
 - ```pdsh -a 'date'```
- RPM count
 - ```pdsh -a 'rpm -qa| wc -l'```
- Kernel version
 - ```pdsh -a uname -a```
- Module count 
 - ```pdsh -a 'lsmod | wc -l'``` 
- Count of mounts
 - ```pdsh -a 'df|wc -l'```


