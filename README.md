Ethernet-Bonding-and-Eucalyptus
===============================

Affected Versions: [All]

Ethernet Bonding is not supported within the Eucalyptus solution. 
Description

The following is provided for community discussion and is not intended for use in a production environment.

Bonding or Link Aggregation addresses two problems, network fault tolerance and increasing bandwidth. There are a number of configuration options to consider.
Eucalyptus Configuration VNET_MODE
   Mangaged, 
   Managed no Vlan
System Kernel, Hypervisor Network Infrastructure
Single switch Multiple switchs
Server Hardware (drivers?)

Many of the hypervisor solution expect a logical bridge interface. The bridge interface may interact poorly with some bond interface configurations. Some bond configurations may have specific server NIC constraints and network infrastructure support.

This guide explores some of the common issues. 
Recommended Bonding Mode

We recommend using Bonding Mode 1, Active-Backup with miitool or ethtool, matching network interfaces and redundant Ethernet switches. This enhances your High Availability with a relatively simple bonding solution.
XEN supported Bonding modes

Active-Backup, (Mode 1) Only one slave in a bond is active. A different slave becomes active if the active slave fails. The bond’s MAC address externally visible on only one port (NIC) to avoid confusing switches.

Balance-slb, (Mode 7) Is an active/active mode, but only supports load-balancing of virtual machine traffic across the physical NICs. Provides fail-over support for all other traffic types. Does not require switch support for Etherchannel or 802.3ad (LACP). Load balances traffic between multiple interfaces at virtual machine granularity by sending traffic through different interfaces based on the source MAC address of the packet. Is derived from the open source ALB mode and reuses the ALB capability to dynamically re-balance load across interfaces. (Reference: http://support.citrix.com/article/CTX124421)
KVM

http://www.linux-kvm.org/page/HOWTO_BONDING http://www.cyberciti.biz/howto/question/static/linux-ethernet-bonding-driver-howto.php
Redhat

http://docs.redhat.com/docs/en-US/Red_Hat_Enterprise_Linux/6-Beta/html/Deployment_Guide/s2-networkscripts-interfaces_network-bridge.html
VMware

http://www.vmware.com/vmtn/resources/412 http://www.vmware.com/resources/techresources/432
Problems and Pitfalls

If Bonding is not forwarding packets correctly, a high packet loss may occur. Symptoms of this may include;NC is very very slow to work on, not sure why. SSHconnection to the host is very slow. There seems to be a problem related to the bonding devices ("received packet with own address as source address")

Recently we were working with a partner and identified a number of issues where libvirt was crashing due to the timing between it starting and the bridge being up. The delay in the bridge was due to the fact that the bond interface was not ready. We saw a number of messages around libvirt crashing in dmesg and /var/log/messages. There are a number of posts around various stability issues with bonded interfaces in bridge mode with libvirt as well.
Active-backup with ARP monitoring on a bond placed in a bridge will simply not work. I would suggest disabling ARP monitoring or using 802.3ad (mode 4) bonding. Any mode where the switch might broadcast frames to an inactive link is one that could cause problems and will ruin the forwarding database in the kernel's bridge. I can confirm that both mode 1 and 5 works with bridge. Mode 6 indeed does not work with bridging due to the ARP problem. Ref: https://bugzilla.redhat.com/sh ow_bug.cgi?id=584872

Problem with Bridge + Bonding. There is a known ARP problem for bridge on a bonded interface. (Ref: https://bugzilla.redhat.com/show_bug.cgi?id=584872 , https://lists.linux-foundation.org/pipermail/bridge/2007-April/005376.htmlManaged Mode, Managed Mode no VLA ) solution: Change to "active-backup" or look change the arp settings on the switch. If the swtich supports "802.11q" or "Dynamic Link Aggregation," mode 4 would also work. Ref:http://www.novell.com/support/kb/doc.php?id=7001989 , https://lists.linux-foundation.org/pipermail/bridge/2007-April/005376.html

Intermittent connectivity on bonded bridges It has been observed that on some low and mid-range switches, bonding mode 0 or "round-robin," network connectivity is intermittent and sparadoc. Some symptoms are: - DomU's may not have any connectivity - Dom0 may not have any connectivity to the network - DomU's and Dom0 are connected cause: Round-robin and a few other modes works by manipulating the ARP table of the switch. Some switches do not work very well with this setting. solution: Change to "active-backup" or look change the arp settings on the switch. If the swtich supports "802.11q" or "Dynamic Link Aggregation," mode 4 would also work. Ref:http://www.novell.com/support/kb/doc.php?id=7001989Solution

50%packet loss when running balance-xor. Switched it back to active-backup and all good

Trouble Shooting

Are all of the parts known to be good?
Disconnect one cable, do the symptoms change?
Reconnect the cable and disconnect the other cable.
Do the symptoms change?
Test each server
Trace the packets ARP cache Bridge tables Network loops?
Broadcast traffic?
Source and destination addresses (MAC and IP)
Packet counts
Recommendations

Matching NICs
Use only known good NICs, cabling and switching equipment
Verify that the bonding solution functions as designed. ( positive and negative testing)
Supporting NIC drivers 
References

Link Aggregation

http://en.wikipedia.org/wiki/Link_aggregation
Centos

http://www.centos.org/docs/4/html/rhel-rg-en-4/s1-networkscripts-interfaces.html 
http://www.centos.org/docs/4/html/rhel-rg-en-4/s1-modules-ethernet.html#S2-MODULES-BONDING 
http://www.cyberciti.biz/tips/linux-bond-or-team-multiple-network-interfaces-nic-into-single-interface.html
Misc.

http://www.fatmin.com/2011/11/ xenserver-switch-ports-configu ration-best-practices.html 
http://en.community.dell.com/t echcenter/enterprise-solutions /w/oracle_solutions/2-1-1-1-ho w-do-i-bond-in-an-oracle-vm-en vironment/revision/1.aspx 
http://support.citrix.com/serv let/KbServlet/download/27046-1 02-666250/XS-design-network_ad vanced.pdf 
http://wiki.xensource.com/xenwiki/XenNetworking


How to configure Network bonding in Linux
http://backdrift.org/howtonetworkbonding 
http://www.linux-kvm.org/page/ HOWTO_BONDING 

Cisco Link Aggregation Control Protocol LACP 8023ad for Gigabit interfaces http://www.cisco.com/en/US/docs/ios/12_2sb/feature/guide/gigeth.html 

Dell
http://www.dell.com/downloads/global/power/ps1q07-20070100-Gautreau-OE.pdf