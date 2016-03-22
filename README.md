# cluster-checks
A collection of checks for cluster homogenity. I'll do some fancy checks at some point but right now it's in a checklist form.

- Operational fans per node
- RPMs of fans
- Operational PDUs
- PDU power draw 
- PDU firmware
- Number of functional temperature sensors 
- Temperatures should be consistent
Disks
- Quick test of disk bandwidth
- Elements in the grown defect list
- Elements in reallocated sector count
- Short self test 
- Geometry of disks
- Number of mounts should be conistent

Management processor
- Event log size
- Unusual event log messages
- Time correct
- Any new messages in the first week of operation? 

Memory
- Total amount of memory

BIOS
- BIOS version
- Output of dmidecode (consistent length, same amount of differences in output between each node)
- dmesg log length after a boot of the whole system and after idling the cluster for at least 3 hours

OS -level
- Clocks synchronized
- RPM count
- Kernel version
- Module count 
