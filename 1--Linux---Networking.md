# 1- Linux - Networking <!-- Metadata: type: Outline; tags: ssh,networking,linux; created: 2020-05-08 13:25:03; reads: 167; read: 2020-05-19 12:33:31; revision: 167; modified: 2020-05-19 12:33:31; importance: 0/5; urgency: 0/5; -->
* [SSH Tunneling](#ssh-tunneling) <kbd>ssh</kbd>
* [Linux Networking configuration](#linux-networking-configuration) <kbd>linux</kbd> <kbd>networking</kbd> <kbd>nmcli</kbd> <kbd>ip</kbd>
    * [Net Status / check](#net-status---check)
    * [nmcli command](#nmcli-command)
    * [Exadata dbnode bounding configuration example](#exadata-dbnode-bounding-configuration-example)
    * [Net Troubleshooting](#net-troubleshooting)
    * [Firewalld](#firewalld)
    * [Oracle Linux 6 - Bonding ](#oracle-linux-6---bonding)
# Linux Networking configuration <!-- Metadata: type: Note; tags: linux,networking,nmcli,ip; created: 2020-05-08 13:25:03; reads: 54; read: 2020-05-19 12:31:36; revision: 15; modified: 2020-05-19 12:31:36; -->


[Networking training oracle internal ... brocken](http://corum.us.oracle.com/acme/network.html)



 

 ### advanced net configuration

[iproute.x86_64 : Advanced IP routing and network device configuration tools](https://blogs.oracle.com/networking/advance-routing-for-multi-homed-hosts)


[How to connect two network interfaces on the same subnet?](https://access.redhat.com/solutions/30564)

[routing-tables](http://linux-ip.net/html/routing-tables.html)

```
 " /etc/sysconfig/network-scripts/route-net2 " , then add the below line. 

172.18.10.0/23 dev net2 table NET2
default via 172.18.10.15 dev net2 table NET2

Then create the below file. 

" /etc/sysconfig/network-scripts/rule-net2 " , then add the below line. 

from 172.18.10.44/23 table NET2
to 172.18.10.44/23 table NET2

Save the file, This will help to retain the customized routes. 
  
```
 
 
## Net Status / check <!-- Metadata: type: Note; created: 2020-05-19 10:48:06; reads: 34; read: 2020-05-19 12:31:43; revision: 6; modified: 2020-05-19 10:56:29; -->

[Which process are listen a port](https://debian-administration.org/article/184/How_to_find_out_which_process_is_listening_upon_a_port)


```
sudo lsof -i :80
sudo lsof -iTCP -sTCP:LISTEN -P -n
sudo lsof -iTCP -sTCP:ESTABLISHED -P -n
sudo lsof -iUDP -P -n | egrep -v '(127|::1)'

sar -n TCP,ETCP,DEV 1
ss -mop
iptraf
```

 
 ## looking for net devices
 
 How to check the network interfaces

* using dmidecode
```
 # dmidecode | egrep -i "PCI|Available|in use"
    PCI is supported
    ESCD support is available
    Designation: PCI-E Slot 1
    Type: x8 PCI Express
    Current Usage: In Use
    Designation: PCI-E Slot 2
    Type: x8 PCI Express
    Current Usage: Available
    Designation: PCI-E Slot 3
    Type: x4 PCI Express
    Current Usage: Available
    Designation: PCI-E Slot 4
    Type: x8 PCI Express
    Current Usage: In Use
    Designation: PCI-E Slot 5
    Type: x8 PCI Express
    Current Usage: In Use

 # dmidecode | grep -i "NIC"
    HP BIOS NIC PCI and MAC Information
    NIC 1: PCI device 04:00.0, MAC address 00:1E:0B:C0:F4:92
    NIC 2: PCI device 42:00.0, MAC address 00:1E:0B:C0:F4:90
    HP BIOS iSCSI NIC PCI and MAC Information
    NIC 1: PCI device 04:00.0, MAC address 00:1E:0B:C0:F4:93
    NIC 2: PCI device 42:00.0, MAC address 00:1E:0B:C0:F4:91	
	
```
* Using ip monitor command

```	
	
$ ip monitor all
[NEIGH]192.168.56.100 dev eth2 lladdr 08:00:27:8a:a8:b7 STALE
[NEIGH]192.168.1.1 dev eth5 lladdr ac:84:c6:05:a5:90 REACHABLE
[NEIGH]192.168.1.1 dev eth5 lladdr ac:84:c6:05:a5:90 STALE
[NEIGH]192.168.1.1 dev eth5 lladdr ac:84:c6:05:a5:90 REACHABLE
[NEIGH]192.168.1.1 dev eth5 lladdr ac:84:c6:05:a5:90 STALE
[NEIGH]192.168.1.1 dev eth5 lladdr ac:84:c6:05:a5:90 REACHABLE
[NEIGH]192.168.1.1 dev eth5 lladdr ac:84:c6:05:a5:90 STALE
[NEIGH]192.168.1.1 dev eth5 lladdr ac:84:c6:05:a5:90 REACHABLE
[NEIGH]192.168.1.1 dev eth5 lladdr ac:84:c6:05:a5:90 STALE
[NEIGH]192.168.1.1 dev eth5 lladdr ac:84:c6:05:a5:90 STALE
[NEIGH]192.168.1.1 dev eth5 lladdr ac:84:c6:05:a5:90 STALE
[NEIGH]192.168.1.1 dev eth5 lladdr ac:84:c6:05:a5:90 REACHABLE

$ ip -s link list eth5
6: eth5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:30:61:95 brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast   
    309687     3511     0       0       0       69      
    TX: bytes  packets  errors  dropped carrier collsns 
    289910     2772     0       0       0       0       

```

* Using lspci and dmesg

```
lspci |grep -i --color ether
04:00.0 Ethernet controller: Intel Corporation 80003ES2LAN Gigabit Ethernet Controller (Copper) (rev 01)
04:00.1 Ethernet controller: Intel Corporation 80003ES2LAN Gigabit Ethernet Controller (Copper) (rev 01)


dmesg |grep 04:00.0
pci 0000:04:00.0: [8086:1096] type 0 class 0x000200
pci 0000:04:00.0: reg 10: [mem 0xdffe0000-0xdfffffff]
pci 0000:04:00.0: reg 18: [io  0xec00-0xec1f]
pci 0000:04:00.0: PME# supported from D0 D3hot D3cold
pci 0000:04:00.0: PME# disabled
pci 0000:04:00.0: disabling ASPM on pre-1.1 PCIe device.  You can enable it with 'pcie_aspm=force'
pci 0000:04:00.0: Signaling PME through PCIe PME interrupt
e1000e 0000:04:00.0: PCI INT A -> GSI 18 (level, low) -> IRQ 18
e1000e 0000:04:00.0: setting latency timer to 64
e1000e 0000:04:00.0: eth0: (PCI Express:2.5GT/s:Width x4) 00:1e:68:a9:d3:8a
e1000e 0000:04:00.0: eth0: Intel(R) PRO/1000 Network Connection
e1000e 0000:04:00.0: eth0: MAC: 6, PHY: 5, PBA No: FFFFFF-0FF
e1000e 0000:04:00.0: eth0: 10/100 speed: disabling TSO


 lspci -vs 04:00.0
04:00.0 Ethernet controller: Intel Corporation 80003ES2LAN Gigabit Ethernet Controller (Copper) (rev 01)
        Subsystem: Oracle Corporation Device 534f
        Flags: bus master, fast devsel, latency 0, IRQ 130
        Memory at dffe0000 (32-bit, non-prefetchable) [size=128K]
        I/O ports at ec00 [size=32]
        Capabilities: [c8] Power Management version 2
        Capabilities: [d0] MSI: Enable+ Count=1/1 Maskable- 64bit+
        Capabilities: [e0] Express Endpoint, MSI 00
        Capabilities: [100] Advanced Error Reporting
        Capabilities: [140] Device Serial Number 00-1e-68-ff-ff-a9-d3-8a
        Kernel driver in use: e1000e
        Kernel modules: e1000e



]# ls -l /sys/class/net/
total 0
lrwxrwxrwx. 1 root root 0 Mar 26 16:04 enp4s0f0 -> ../../devices/pci0000:00/0000:00:09.0/0000:02:00.0/0000:03:02.0/0000:04:00.0/net/enp4s0f0
lrwxrwxrwx. 1 root root 0 Mar 26 16:04 enp4s0f1 -> ../../devices/pci0000:00/0000:00:09.0/0000:02:00.0/0000:03:02.0/0000:04:00.1/net/enp4s0f1
lrwxrwxrwx. 1 root root 0 Mar 26 16:04 enp9s0f0 -> ../../devices/pci0000:00/0000:00:01.0/0000:09:00.0/net/enp9s0f0
lrwxrwxrwx. 1 root root 0 Mar 26 16:04 enp9s0f1 -> ../../devices/pci0000:00/0000:00:01.0/0000:09:00.1/net/enp9s0f1
lrwxrwxrwx. 1 root root 0 Mar 26 16:04 enp9s0f2 -> ../../devices/pci0000:00/0000:00:01.0/0000:09:00.2/net/enp9s0f2
lrwxrwxrwx. 1 root root 0 Mar 26 16:04 enp9s0f3 -> ../../devices/pci0000:00/0000:00:01.0/0000:09:00.3/net/enp9s0f3
lrwxrwxrwx. 1 root root 0 Mar 26 16:04 lo -> ../../devices/virtual/net/lo
lrwxrwxrwx. 1 root root 0 Mar 26 16:04 virbr0 -> ../../devices/virtual/net/virbr0
lrwxrwxrwx. 1 root root 0 Mar 26 16:04 virbr0-nic -> ../../devices/virtual/net/virbr0-nic

```



 ### uuidgen eth0
```
/etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
BOOTPROTO=none
UUID=
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1   ---> put this only for the default router network
HWADDR=00:1E:0B:C0:F4:93
PREFIX=24
DEFROUTE=yes

 more /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE="eth0"
NM_CONTROLLED="yes"
ONBOOT="yes"
TYPE=Ethernet
BOOTPROTO=dhcp
IPV4_FAILURE_FATAL=no
IPV6INIT=no
NAME=Nat-eth0
UUID=5fb06bd0-0bb0-7ffb-45f1-d6edd65f3e03
DEFROUTE=no
HWADDR=08:00:27:A7:DA:CB
PEERDNS=yes
PEERROUTES=no

 # more /etc/sysconfig/network-scripts/ifcfg-eth1
DEVICE="eth1"
NM_CONTROLLED="yes"
ONBOOT="yes"
TYPE=Ethernet
BOOTPROTO=none
IPV4_FAILURE_FATAL=no
IPV6INIT=no
NAME=public-eth1
DEFROUTE=yes
NETMASK=255.255.255.0
IPADDR=192.168.0.126
GATEWAY=192.168.0.1
PREFIX=24
DNS1=192.168.0.1
UUID=9c92fad9-6ecb-3e6c-eb4d-8a47c6f50c04
LAST_CONNECT=1519064513
```

 ### speed 10GB no auto neg 

```
echo 'ETHTOOL_OPTS="speed 10000 duplex full autoneg off"' >> /etc/sysconfig/network-scripts/ifcfg-eth1
```
### Network Issus and toubleshooting
  <!-- Metadata: type: Note; created: 2020-05-19 10:56:53; reads: 37; read: 2020-05-19 12:31:06; revision: 7; modified: 2020-05-19 12:30:59; -->

 ## 1. Error: Connection activation failed: Master connection not found or invalid

You need add this parameter to the NM_CONTROLLED=no to the connfiguration file


GUIDE: Oracle VM VirtualBox and Oracle Linux NIC bonding [Guide](https://community.oracle.com/thread/2546040)
fail_over_mac=1

```
cat /etc/sysconfig/network-scripts/ifcfg-bond0
DEVICE=bond0
IPADDR=10.0.2.12
NETMASK=255.0.0.0
GATEWAY=10.0.0.138
DNS1=10.0.0.138
DNS2=8.8.8.8
ONBOOT=yes
BOOTPROTO=none
USERCTL=no
BONDING_OPTS="mode=1 miimon=100 fail_over_mac=1"

/etc/modprobe.d/bonding-eth0_eth1.conf
alias bond0 bonding
```

 ## 2. Intermittent Slower Network and Connection Timeouts (Doc ID 1614134.1)	

  Attempts to connect to a server with ssh and/or sftp results in time-outs or a delayed response.
  
When the network load goes high, there are high number of network retransmits observed.


```
The output of command : netstat -s shows increasing values for the following stats 
:(run several times of 'netstat -s')


13336 packets pruned from receive queue because of socket buffer overrun
516 times the listen queue of a socket overflowed
516 SYNs to LISTEN sockets ignored
2040077 packets collapsed in receive queue due to low socket buffer
TCPBacklogDrop: 744165
  
The output of command : 

ethtool -S eth[x]  
shows increasing values against the counter : "rx_fw_discards"
rx_fw_discards: 4493
There may be large number of packet drops seen in the output of " ifconfig eth[x] " command.

find . -name 'ethtool_-S_*' -type f -exec grep -i rx_fw_discards {} \; -print
find . -name 'netstat_-s' -type f -exec grep -i TCPBacklogDrop {} \; -printf '%TY-%Tm-%Td %TT %p\n'
```



 ## 3. package drop analysis

[Netstat command](https://www.ibm.com/developerworks/community/blogs/58e72888-6340-46ac-b488-d31aa4058e9c/entry/linux_netstat_command_explained_with_10_examples?lang=en)

[iface netstat](https://www.tldp.org/LDP/nag2/x-087-2-iface.netstat.html)

```
$ netstat -i
Kernel Interface table
Iface   MTU Met   RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
eth0       1500 0         0      0      0 0             0      0      0      0 BMU
lo        16436 0        44      0      0 0            44      0      0      0 LRU
wlan0      1500 0    166164      0      0 0        152434      0      0      0 BMRU

RX-OK   : Correct packets received on this interface. 
RX-ERR : Incorrect packets received on this interface 
RX-DRP : Packets that were dropped at this interface. 
RX-OVR : Packets that this interface was unable to receive. 

$ netstat -r
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
192.168.1.0     *               255.255.255.0   U         0 0          0 wlan0
link-local      *               255.255.0.0     U         0 0          0 wlan0
default         192.168.1.1     0.0.0.0         UG        0 0          0 wlan0

A Receive all multicast at this interface.
B OK broadcast.
D Debugging ON. 
M Promiscuous Mode.
O No ARP at this interface.
P P2P connection at this interface. 
R Interface is running. 
U Interface is up. 
G Not a direct entry.


ethtool -S eth1-04
NIC statistics:
     rx_packets: 362103638440
     tx_packets: 126303479177
     rx_bytes: 294503297876960
     tx_bytes: 61213346090829
     rx_broadcast: 4028
     tx_broadcast: 654446
     rx_multicast: 67084922
     tx_multicast: 1958037
     multicast: 67084922
     collisions: 0
     rx_crc_errors: 602    ---> Packets are checked for integrety by cyclic redundancy check ( CRC ), when this check fails the rx_crc_errors counters increase
     rx_no_buffer_count: 33786
     rx_missed_errors: 1151959
     tx_aborted_errors: 0
     tx_carrier_errors: 0
     tx_window_errors: 0
     tx_abort_late_coll: 0
(...)
https://linux.die.net/man/8/ethtool


--
 LOOP 1.
 zzz <02/25/2019 23:57:29> subcount: 0
Kernel Interface table
Iface MTU Met RX-OK RX-ERR RX-DRP RX-OVR TX-OK TX-ERR TX-DRP TX-OVR Flg
bond0 1500 0 0 0 0 0 0 0 0 0 BMm
bondeth0 1500 0 13270377448 1 6824874 0 27932004977 0 2 0 BMmRU <<<<<<<<<<<<<<<< RX-ERR=1 RX-DRP=6.824.874 || TX-ERR=0 TX-DR=2
bondeth0:1 1500 0 - no statistics available - BMmRU
bondeth0:2 1500 0 - no statistics available - BMmRU
eth0 1500 0 37649311 0 39379 0 18319454 0 0 0 BMRU
eth1 1500 0 6475147839 0 0 0 6702698062 0 0 0 BMsRU
eth2 1500 0 6795234986 1 0 0 21229333807 0 0 0 BMsRU
eth3 1500 0 0 0 0 0 0 0 0 0 BMU
eth4 1500 0 0 0 0 0 0 0 0 0 BM	
eth5 1500 0 0 0 0 0 0 0 0 0 BM
ib0 7000 0 45978028 0 0 0 37350452 0 21 0 BMRU
ib0:P02 7000 0 - no statistics available - BMRU
ib0:1 7000 0 - no statistics available - BMRU
ib0:2 7000 0 - no statistics available - BMRU
ib1 7000 0 0 0 0 0 0 0 0 0 BMU
lo 65536 0 6569275918 0 0 0 6569275918 0 0 0 LRU
Ip:

It means just 606.057 received packages dropped (RX-DRP) beteween 02/25/2019 23:57:29 and 02/28/2019 10:24:02.
I dont think that is the issue. -This package dropping This possibly an "designed" behavior, please check :
The Statistic Value of Dropped Packets of Network Interface Increases Constantly in UEK2/UEK3/UEK4 Kernel (Doc ID 1515650.1)
```



 ## 4.  rsyc fail to transfer 

 Connection using SSH to a Host Not in DNS/hosts Stalls for Some Time at Connection Initiation (Doc ID 1066024.1)	
	Scp Transfer 'stall' Between Oracle Linux Servers (Doc ID 1997673.1)	

	rsync -rtvhu --progress /backup/Sincro/AYR21 172.20.115.197:/backup/Sincro
	sending incremental file list
	AYR21/control.ctl
	Write failed: Broken pipekB/s    0:00:00

	rsync error: unexplained error (code 255) at log.c(235) [sender=3.0.6]
	[oracle@FIDUORADB01 AYR21]$CRONT

 * VALIDAR CUANDO NO FUNCIONA EL RSYN O SCP NO PERMITEN PASAR ARCHIVOS MAYORES A 1M


[rsync/scp transfers stall quickly between 2 hosts](https://access.redhat.com/solutions/4384121)

```
cat /etc/sysctl.conf    ----------  debe ser 0, se cambia asi
sysctl -w net.ipv4.tcp_sack=0
cat /proc/sys/net/ipv4/tcp_mtu_probing 
echo 2 > /proc/sys/net/ipv4/tcp_mtu_probing
```

 ## 5. rp_filter
  [rp_filter for multiple private interconnects and Linux Kernel 2.6.32+ (Doc ID 1286796.1)](https://support.oracle.com/oip/faces/secure/km/DocumentDisplay.jspx?id=2021771.1)

[lartc.kernel.rpf](https://tldp.org/HOWTO/Adv-Routing-HOWTO/lartc.kernel.rpf.html)
	The rp_filter parameter can be set globally on all NICs, or specific on each NIC. The maximum (between the "all" and the specific interface) value is taken. The values are defined as follows:

```
  rp_filter - INTEGER

 0 - No source validation.
 1 - Strict mode as defined in RFC3704 Strict Reverse Path Each incoming packet is tested against the FIB and if the interface is not the best reverse path the packet check will fail. By default failed packets are discarded.
 2 - Loose mode as defined in RFC3704 Loose Reverse Path Each incoming packet's source address is also tested against the FIB and if the source address is not reachable via any interface the packet check will fail.

```

 ## 6. NFS Debug 

```
==== ACTION PLAN To collect ** during time frame of the issue** ====

1. To enable debugging (NFS server side)
 # rpcdebug -m nfsd -s all
 # rpcdebug -m nfsd -s lockd 


2. To enable debugging (NFS client side)
 #rpcdebug -m nfs -s all

3) Collect a tcpdump from the NFS client to the NFS server.
 # tpcdump -i <interface> -w /tmp/capture.pcap host <NFS_SERVER>

NOTE= has the tcpdump running on a terminal until the test end.

4) Replicate the issue, copying data as the customer did before
When the test finish.

4) Stop the kernel debug messages (NFS client side)
 # rpcdebug -m nfs -c all

5) Stop the kernel debug messages (NFS server side)
 # rpcdebug -m nfsd -c all

6) Stop the tcpdump

7) Provide the /var/log/messages of client and server server and the generated /tmp/script1.log file of the client node.

More info of test.
- How to debug NFS/RPC status via rpcdebug (Doc ID 2259249.1)

-----------------------------


```
### Networking Performance <!-- Metadata: type: Note; created: 2020-05-19 12:31:55; reads: 6; read: 2020-05-19 12:33:21; revision: 3; modified: 2020-05-19 12:33:21; -->

[MEASURING NETWORK PERFORMANCE IN LINUX WITH QPERF](https://www.opsdash.com/blog/network-performance-linux.html)

[How to use qperf to measure network bandwidth and latency performance?](https://access.redhat.com/solutions/2122681)


 ### Docs IDs 

* 	[Connection using SSH to a Host Not in DNS/hosts Stalls for Some Time at Connection Initiation (Doc ID 1066024.1)](https://support.oracle.com/oip/faces/secure/km/DocumentDisplay.jspx?id=2022086.1)
* Large receive offload (LRO) and Generic Receive Offload (GRO) 

* 7.81 GRO Must Be Disabled After an Upgrade When Routing/Bridging with IXGBE Driver  [vmrns-bugs](https://docs.oracle.com/cd/E64076_01/E64077/html/vmrns-bugs-3.3.3-21353855.html)



https://www.kernel.org/doc/Documentation/sysctl/net.txt
/etc/sysctl.conf

 ### dev_weight

The maximum number of packets that kernel can handle on a NAPI interrupt, it's a Per-CPU variable. For drivers that support LRO or GRO_HW, a hardware aggregated packet is counted as one packet in this context.
Default: 64

[Large_receive_offload](https://en.wikipedia.org/wiki/Large_receive_offload)

[how-to-enable-large-receive-offload--lro-x](https://community.mellanox.com/s/article/how-to-enable-large-receive-offload--lro-x)

[NICs with RX acceleration (GRO/LRO/TPA/etc) may suffer from bad TCP performance](https://access.redhat.com/solutions/20278)


[Generic Receive Offload Library](https://doc.dpdk.org/guides/prog_guide/generic_receive_offload_lib.html)


Generic Receive Offload (GRO) is a widely used SW-based offloading technique to reduce per-packet processing overheads. By reassembling small packets into larger ones, GRO enables applications to process fewer large packets directly, thus reducing the number of packets to be processed. To benefit DPDK-based applications, like Open vSwitch, DPDK also provides own GRO implementation. In DPDK, GRO is implemented as a standalone library. Applications explicitly use the GRO library to reassemble packets.


 ## 2.  network_throughput_test.sh Script

[Zero Data Loss Recovery Appliance Network Test Throughput script (Doc ID 2022086.1)](https://support.oracle.com/oip/faces/secure/km/DocumentDisplay.jspx?id=2022086.1)


	PURPOSE
	This troubleshooting note aims to identify the theoretical network throughput between a Protected Database system and a Zero Data Loss Recovery Appliance, or between two Zero Data Loss Recovery Appliances in a replication configuration.

	TROUBLESHOOTING STEPS
	Assumptions
	This script uses the QPERF utility that is provided as part of an Exadata Engineered System as well as part of the Zero Data Loss Recovery Appliance.

	This script may be executed from any Linux server with a BASH shell, and requires trusted SSH to be established between the execution node and the two sets of nodes that are being tested.

	The Execution node may be one of the nodes being tested.

	The O/S user used for trusted SSH is the O/S user running the script. There is no support for the script to use a different O/S user for logging into the nodes being tested.

	Installation
	If not already available under /opt/oracle.RecoveryAppliance/client then download the script attached to this MOS Note and copy it to the desired location.


```
	Usage
	$ ./network_throughput_test.sh -h
	Usage: ./network_throughput_test.sh -s [sending_nodes_file] -r [receiving_nodes_file] -i [interface]
	Parameters
	The files "sending_nodes_files" and "receiving_nodes_file" are in the format of one hostname per line, similar to the group file used by DCLI utility

	The "interface" defaults to "bondeth0" but can be any interface on the receiving nodes, such as bondeth1 if testing network throughput to a downstream replica.

	Example
	$ ./network_throughput_test.sh -s sending_nodes -r receiving_nodes -i bondeth0
	Using 'scam09db01 scam09db02 scam09db03 scam09db04 scam09db05 scam09db06 scam09db07 scam09db08' hosts for sending nodes
	Using 'scas10adm01 scas10adm02 scas10adm03 scas10adm04' hosts for receiving nodes
	Validating Trusted SSH to sending nodes scam09db01 scam09db02 scam09db03 scam09db04 scam09db05 scam09db06 scam09db07 scam09db08 ... OK
	Validating Trusted SSH to receiving nodes scas10adm01 scas10adm02 scas10adm03 scas10adm04 ... OK
	Total Network Bandwidth 8,372,597,121 bytes/sec
```
## nmcli command <!-- Metadata: type: Note; created: 2020-05-19 10:38:05; reads: 20; read: 2020-05-19 12:26:28; revision: 3; modified: 2020-05-19 10:38:12; -->

 ## nmcli 
 
 ### nmcli examples

[configure-network-connections-using-nmcli-tool-in-linux](https://www.tecmint.com/configure-network-connections-using-nmcli-tool-in-linux/)

[nmcli-examples ](https://people.freedesktop.org/~lkundrak/nm-docs/nmcli-examples.html)

[Using_the_NetworkManager_Command_Line_Tool_nmcli](https://docs.fedoraproject.org/en-US/Fedora/21/html/Networking_Guide/sec-Using_the_NetworkManager_Command_Line_Tool_nmcli.html)

 

```
CLIENT MAC ADDR: 00 26 9E B5 CC 96  GUID: 080020FF FFFF FFFF FFFF 00269EB5CC96
nmcli con mod enp6s0f1 ipv4.gateway "10.145.236.1"
nmcli con down enp6s0f1
nmcli con up enp6s0f1

nmcli con mod enp6s0f0 ipv4.gateway ""
nmcli con add type ethernet con-name enp6s0f0 ifname enp6s0f0 ip4 10.10.10.178/24 gw4 10.10.10.253 
nmcli connection modify enp6s0f0 +ipv4.routes "10.10.10.178/24 10.10.10.253"



nmcli con add type ethernet con-name enp6s0f1 ifname enp6s0f1 ip4 10.145.237.250/23 gw4 10.145.236.1
nmcli connection modify enp6s0f1 +ipv4.routes "10.145.237.250/23 10.145.236.1"
nmcli con mod enp6s0f1 ipv4.dns "206.223.27.1 144.20.190.70"

nmcli con down enp6s0f1
nmcli con up enp6s0f1


 # ip route
default via 10.145.236.1 dev enp6s0f1 proto static metric 101
10.10.10.0/24 dev enp6s0f0 proto kernel scope link src 10.10.10.178 metric 100
10.10.10.0/24 via 10.10.10.253 dev enp6s0f0 proto static metric 100
10.145.236.0/23 dev enp6s0f1 proto kernel scope link src 10.145.237.250 metric 101
10.145.236.0/23 via 10.145.236.1 dev enp6s0f1 proto static metric 101
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown

ip route add 10.10.254.0/29 via 10.10.254.1 dev eth1

------------------------------- 
 ```

 ### nmcli command - This works only in Redhat 7 and Oracle linux 7   --- it will change from version 7.1 to 7.2*

[rhel7 configure ipv4 addresses](https://www.certdepot.net/rhel7-configure-ipv4-addresses/)

```
nmcli dev status
DEVICE  TYPE      STATE         CONNECTION
eth0    ethernet  connected     eth0
eth1    ethernet  connected     eth1
eth2    ethernet  disconnected  --
eth3    ethernet  disconnected  --
lo      loopback  unmanaged     --

nmcli dev show


nmcli con add type ethernet con-name eth2_1 ifname eth2 ip4 192.168.1.50/24 gw4 192.168.1.1  ---> static IP


nmcli con mod eth2_1 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli con mod eth2_1 ipv4.addresses "192.168.100.100/24"
nmcli con mod eth2_1 ipv4.gateway "192.168.100.1"
nmcli con mod eth2_1 connection.autoconnect on
nmcli con mod eth2_1 ipv4.method manual  ---> to change from dhcp to manual
nmcli con up eth2_1
nmcli con show eth2_1
nmcli con down eth2_1
nmcli con del eth2_1  ---> remove the connection



nmcli con add type ethernet con-name eno1 ifname eno1 ip4 192.168.1.3/24 gw4 192.168.1.1 
nmcli con up eno1
```


 ### IP config

```
ip addr
ip -4 a s

ip addr show  | grep 'inet ' | awk '{print $2}' | cut -f1 -d'/'
127.0.0.1
10.10.10.178
10.145.237.250
192.168.122.1

```

 ### To know the default gateway
```
nmcli --terse --fields IP4.GATEWAY device show ${1} | cut -f2- -d':'
10.145.236.1 
```


 ### DCHP 

```
nmcli con add type ethernet con-name eth2_1 ifname eth2  ----> dhcp
nmcli con mod eth2_1 ipv4.method manual  ---> to change from dhcp to manual
nmcli con mod eth2_1 ipv4.method auto  ---> to change from manual to dhcp 

nmcli con add type ethernet con-name enp0s17-bridge ifname enp0s17 autoconnect yes save yes


nmcli con mod  "Wired connection 1" enp0s3-bridge
```

### bond config with nmcli
```
nmcli con add type bond con-name bond0 ifname bond0 mode active-backup ip4 192.168.1.201/24
nmcli con add type bond-slave ifname eth2 master bond0
nmcli con add type bond-slave ifname eth3 master bond0
```

* [Bonding real customer LACP mode 4 with OVM server](https://developer.rackspace.com/blog/lacp-bonding-and-linux-configuration/)

```
$ more ifcfg-bond0
DEVICE=bond0
BONDING_OPTS="mode=4 miimon=250 use_carrier=1 updelay=500 downdelay=500 primary_reselect=2 primary=eth0"
BOOTPROTO=none
ONBOOT=yes

$ more ifcfg-eth0
DEVICE="eth0"
BOOTPROTO=none
HWADDR="00:10:E0:62:F4:5E"
ONBOOT="yes"
MASTER=bond0
SLAVE=yes

$ cat proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: IEEE 802.3ad Dynamic link aggregation
Transmit Hash Policy: layer2 (0)
MII Status: up
MII Polling Interval (ms): 250
Up Delay (ms): 500
Down Delay (ms): 500

802.3ad info
LACP rate: slow
Min links: 0
Aggregator selection policy (ad_select): stable
Active Aggregator Info:
        Aggregator ID: 1
        Number of ports: 1
        Actor Key: 33
        Partner Key: 1
        Partner Mac Address: 00:00:00:00:00:00

Slave Interface: eth0
MII Status: up
Speed: 10000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 00:10:e0:62:f4:5e
Aggregator ID: 1
Slave queue ID: 0
```
## Exadata dbnode bounding configuration <!-- Metadata: type: Note; created: 2020-05-19 10:58:15; reads: 7; read: 2020-05-19 10:59:21; revision: 3; modified: 2020-05-19 10:59:21; -->


 ### Exadata example

```
27Nov2017-03:55# cat /etc/sysconfig/network-scripts/ifcfg-bondeth0
 #### DO NOT REMOVE THESE LINES ####
 #### %GENERATED BY CELL% ####
DEVICE=bondeth0
USERCTL=no
BOOTPROTO=none
ONBOOT=yes
BONDING_OPTS="mode=active-backup miimon=100 downdelay=5000 updelay=5000 num_grat_arp=100"
IPV6INIT=no
IPADDR=192.168.242.150
NETMASK=255.255.255.0
GATEWAY=192.168.242.11
NETWORK=192.168.242.0
BROADCAST=192.168.242.255
 


ip addr show
cd /sys/class/net
cat /proc/net/bonding/bond0


```
## Firewalld <!-- Metadata: type: Note; created: 2020-05-19 10:37:16; reads: 19; read: 2020-05-19 12:26:26; revision: 4; modified: 2020-05-19 10:59:43; -->


```
firewall-cmd --state
firewall-cmd --get-zones
sudo firewall-cmd --list-all-zones
sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp6s0f0 enp6s0f1
  sources:
  services: dhcpv6-client http https ssh vnc-server
  ports: 8080/tcp 7002/tcp 123/udp 10000/tcp 5903/tcp 5902/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

firewall-cmd --list-ports --zone=public
8080/tcp 7002/tcp 123/udp 10000/tcp 5903/tcp

sudo firewall-cmd --zone=public --permanent --add-service=https
sudo firewall-cmd --zone=public --permanent --add-service=http
sudo firewall-cmd --zone=public --list-services
dhcpv6-client http https ssh
sudo firewall-cmd --reload

sudo firewall-cmd --zone=public --permanent --add-port=1521/tcp
sudo firewall-cmd --zone=public --permanent --add-port=5902/tcp
sudo firewall-cmd --zone=public --permanent --add-port=5903/tcp
sudo firewall-cmd --reload


firewall-cmd --zone=public --permanent --remove-service=http
firewall-cmd --zone=public --permanent --remove-service=https
firewall-cmd --reload

```
## Oracle Linux 6 - Bonding  <!-- Metadata: type: Note; created: 2020-05-19 10:53:45; reads: 15; read: 2020-05-19 12:26:26; revision: 4; modified: 2020-05-19 10:54:44; -->

```
cat /usr/share/doc/iputils-*/README.bonding

OVMx86
BONDING_OPTS="mode=1 miimon=250 use_carrier=1 updelay=500 downdelay=500 primary_reselect=2 primary=eth0 "



/etc/sysconfig/network-scripts/ifcfg-bond0
DEVICE=bond0
BONDING_OPTS="mode=active-backup miimon=250 use_carrier=1 updelay=500 downdelay=500 primary=eth1"
ONBOOT=yes
BOOTPROTO=none
NETMASK=255.255.255.0
IPADDR=192.168.0.126
GATEWAY=192.168.0.1
PREFIX=24
DEFROUTE=yes
USERCTL=no
NM_CONTROLLED=no

/etc/sysconfig/network-scripts/ifcfg-eth1
DEVICE=eth1
ONBOOT=yes
BOOTPROTO=none
MASTER=bond0
SLAVE=yes


Configure the kernel module
* Add the following line to /etc/modprobe.d/bonding.conf file:
	– alias bond0 bonding
* Then execute the following two commands:
	– # modprobe bond0 
	– # service network restart
* To identify which interface is active vs backup, look in;
	– /sys/class/net/bond0/bonding/active_slave

Note: To remove BONDING, #rmmod bonding

Alternative, ifenslave
* To create a bonded device by using the ifenslave command:
– Load the bonding module:
 # modprobe bonding
– Configure the network settings for the bonded interface:
 # ip addr add 192.168.1.221/24 dev bond0
– Bring up the bonded interface:
 # ip link set bond0 up
– Attach the component network interfaces to the bonded interface:
 # ifenslave bond0 eth1 eth3


You can add all the IPs that you want to this bond0 
ip addr add 192.168.1.126/24 dev bond0

6: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP
    link/ether 08:00:27:1c:cb:24 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.221/24 scope global bond0
       valid_lft forever preferred_lft forever
    inet 192.168.1.126/24 scope global secondary bond0
       valid_lft forever preferred_lft forever

 # more /etc/sysconfig/network-scripts/route-eth1
ADDRESS0=192.168.1.0
NETMASK0=255.255.255.0
GATEWAY0=192.168.1.1
METRIC0=1

```
## Virtual network  <!-- Metadata: type: Note; created: 2020-05-19 12:26:08; reads: 13; read: 2020-05-19 12:27:43; revision: 4; modified: 2020-05-19 12:27:43; -->

[libvirt.org](https://libvirt.org/)

[Virtual Networking](http://wiki.libvirt.org/page/VirtualNetworking)

 libvirt is an open source API, daemon and management tool for managing platform virtualization.[3] It can be used to manage KVM, Xen, VMware ESX, QEMU and other virtualization technologies. These APIs are widely used in the orchestration layer of hypervisors in the development of a cloud-based solution.



* brctl - ethernet bridge administration  [man page](https://linux.die.net/man/8/brctl)


 ### Creating a Bridge virtual interface 

[wiki Bridged networking libvirt](https://wiki.libvirt.org/page/Networking#Bridged_networking_.28aka_.22shared_physical_device.22.29)

[fedora-22-libvirt-with-bridge](https://lukas.zapletalovi.com/2015/09/fedora-22-libvirt-with-bridge.html)

[linux-bridge-isolation](https://vincent.bernat.ch/en/blog/2017-linux-bridge-isolation)


```
[root@ovmserver01]# more ifcfg-bond0
 #This file was dynamically created by OVM manager. Please Do not edit
DEVICE=bond0
BOOTPROTO=none
ONBOOT=yes
BRIDGE=0a0a0a00
NM_CONTROLLED=no
BONDING_OPTS="mode=1 miimon=250 use_carrier=1 updelay=500 downdelay=500 primary=eth0"


[root@ovmserver01]# more ifcfg-0a0a0a00
 #This file was dynamically created by OVM manager. Please Do not edit
DEVICE=0a0a0a00
TYPE=Bridge
IPADDR=10.10.10.136
NETMASK=255.255.255.0
BOOTPROTO=static
ONBOOT=yes
DELAY=0

[root@ovmserver01]# more ifcfg-eth0
 # Intel Corporation 80003ES2LAN Gigabit Ethernet Controller (Copper)
DEVICE=eth0
BOOTPROTO=none
HWADDR=00:1E:68:A9:D3:8A
ONBOOT=yes
MASTER=bond0
SLAVE=yes


 # brctl show
bridge name     bridge id               STP enabled     interfaces
0a0a0a00        8000.001e68a9d38a       no              bond0
````
# SSH Tunneling <!-- Metadata: type: Note; tags: ssh; created: 2020-05-10 12:22:34; reads: 27; read: 2020-05-19 12:33:31; revision: 20; modified: 2020-05-10 15:14:13; -->

[Using SSH tunnels with Oracle Linux 8 - youtube video](https://www.youtube.com/watch?time_continue=25&v=NRL_wXqnQeo&feature=emb_logo)

[Ubuntu OpenSSH PortForwarding](https://help.ubuntu.com/community/SSH/OpenSSH/PortForwarding)

Introduction. Port forwarding via SSH (SSH tunneling) creates a secure connection between a local computer and a remote machine through which services can be relayed.

Because the connection is encrypted, SSH tunneling is useful for transmitting information that uses an unencrypted protocol, such as IMAP, VNC, or IRC

There are three types of port forwarding with SSH:

1. **Local port forwarding:** connections from the SSH client are forwarded via the SSH server, then to a destination server

2. **Remote port forwarding:** connections from the SSH server are forwarded via the SSH client, then to a destination server

3. **Dynamic port forwarding:** connections from various programs are forwarded via the SSH client, then via the SSH server, and finally to several destination servers



 ## 1. ssh for Local port Forwarding

Local port forwarding lets you connect from your local computer a ssh server.


1. Using ssh to forward the port 5901 (VNC Server) that is filtered by middle firewall   in the Server side denying the 

* From client ssh server type:
``` 
ssh -L127.0.0.1:5901:10.10.10.178:5901 -l root 10.145.237.250

-L : local 
localhost:5901 : VCN port in localhost
10.10.10.178:5901 : the remote server that are listing the vcn in port 5901
```

 ## 2. ssh remote port forwarding



 ## 3. ssh dynamic port forwarding (socks5)

[SSH socks5 proxy](https://ma.ttias.be/socks-proxy-linux-ssh-bypass-content-filters/)

[How To Route Web Traffic Securely Without a VPN Using a SOCKS Tunnel](https://www.digitalocean.com/community/tutorials/how-to-route-web-traffic-securely-without-a-vpn-using-a-socks-tunnel)

[How to Set up SSH SOCKS Tunnel for Private Browsing](https://linuxize.com/post/how-to-setup-ssh-socks-tunnel-for-private-browsing/)

```
ssh -N -D 9090 [USER]@[SERVER_IP]
The options used are as follows:

-N - Tells SSH not to execute a remote command.
-D 9090 - Opens a SOCKS tunnel on the specified port number.
[USER]@[SERVER_IP] - Your remote SSH user and server IP address.
To run the command in the background use the -f option.
If your SSH server is listening on a port other than 22 (the default) use the -p [PORT_NUMBER] option.
```

* For example you can config the yum application to use the socks5.

```
Edit /etc/yum.conf to include the following configuration information in the content:

proxy=socks5://10.10.10.178:8080
#proxy_username=username
#proxy_password=password
```
