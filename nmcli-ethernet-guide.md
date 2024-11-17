# RHEL 7 nmcli Ethernet Device Management Cheat Sheet

## Basic Device Information Commands

| Command | Description |
|---------|-------------|
| `nmcli device status` | Show status of all network interfaces |
| `nmcli device show` | Display detailed information about all devices |
| `nmcli device show eth0` | Show detailed information for specific interface |
| `nmcli connection show` | List all network connections |
| `nmcli connection show --active` | List only active connections |

## Creating New Ethernet Connections

| Command | Description |
|---------|-------------|
| `nmcli connection add type ethernet con-name eth0 ifname eth0` | Create basic ethernet connection |
| `nmcli con add type ethernet con-name eth0 ifname eth0 ip4 192.168.1.100/24 gw4 192.168.1.1` | Create connection with static IP |
| `nmcli con add type ethernet con-name eth0 ifname eth0 ipv4.method auto` | Create connection with DHCP |
| `nmcli con add type ethernet con-name eth0 ifname eth0 ip4 192.168.1.100/24 gw4 192.168.1.1 ip4.dns "8.8.8.8,8.8.4.4"` | Create connection with static IP, gateway and DNS |

## Modifying Existing Connections

| Command | Description |
|---------|-------------|
| `nmcli con mod eth0 ipv4.addresses "192.168.1.100/24"` | Change IP address |
| `nmcli con mod eth0 ipv4.gateway "192.168.1.1"` | Change gateway |
| `nmcli con mod eth0 ipv4.dns "8.8.8.8,8.8.4.4"` | Change DNS servers |
| `nmcli con mod eth0 ipv4.method manual` | Change to static IP configuration |
| `nmcli con mod eth0 ipv4.method auto` | Change to DHCP configuration |
| `nmcli con mod eth0 connection.autoconnect yes` | Enable auto-connect |
| `nmcli con mod eth0 connection.autoconnect no` | Disable auto-connect |

## Managing Connection State

| Command | Description |
|---------|-------------|
| `nmcli con up eth0` | Activate a connection |
| `nmcli con down eth0` | Deactivate a connection |
| `nmcli con reload` | Reload all connection files |
| `nmcli device reapply eth0` | Reapply connection settings |
| `nmcli device disconnect eth0` | Disconnect interface |
| `nmcli device connect eth0` | Connect interface |

## Advanced Configuration

| Command | Description |
|---------|-------------|
| `nmcli con mod eth0 802-3-ethernet.mtu 9000` | Change MTU size |
| `nmcli con mod eth0 ethernet.mac-address AA:BB:CC:DD:EE:FF` | Set MAC address |
| `nmcli con mod eth0 ipv4.never-default true` | Prevent as default route |
| `nmcli con mod eth0 connection.interface-name eth0` | Change interface name |
| `nmcli con mod eth0 ipv4.dns-search "example.com"` | Set DNS search domain |

## Deleting Connections

| Command | Description |
|---------|-------------|
| `nmcli con delete eth0` | Delete a connection profile |
| `nmcli con delete uuid UUID` | Delete connection by UUID |

## Troubleshooting Commands

| Command | Description |
|---------|-------------|
| `nmcli device wifi rescan` | Rescan for available networks |
| `nmcli general status` | Show NetworkManager status |
| `nmcli networking on/off` | Enable/disable networking |
| `nmcli con load /etc/sysconfig/network-scripts/ifcfg-eth0` | Load connection from file |
| `nmcli device set eth0 managed yes` | Set device as managed by NetworkManager |

## Important Notes:
1. Always backup network configuration files before making changes
2. Use `nmcli con reload` after manual changes to configuration files
3. Some changes may require connection restart to take effect
4. Use tab completion with nmcli for command syntax help
5. Connection names are case-sensitive

# LACP (802.3ad) Bond Configuration

## Creating LACP Bond Interface

| Command | Description |
|---------|-------------|
| `nmcli con add type bond con-name bond0 ifname bond0 bond.options "mode=802.3ad,miimon=100,downdelay=200,updelay=200,lacp_rate=1,xmit_hash_policy=layer3+4"` | Create bond interface with LACP parameters |

## Adding Slave Interfaces to LACP Bond

| Command | Description |
|---------|-------------|
| `nmcli con add type bond-slave ifname eth0 master bond0` | Add eth0 as slave to bond0 |
| `nmcli con add type bond-slave ifname eth1 master bond0` | Add eth1 as slave to bond0 |

## Configuring IP for LACP Bond Interface

| Command | Description |
|---------|-------------|
| `nmcli con mod bond0 ipv4.addresses "192.168.1.100/24"` | Set static IP for bond0 |
| `nmcli con mod bond0 ipv4.gateway "192.168.1.1"` | Set gateway for bond0 |
| `nmcli con mod bond0 ipv4.method manual` | Set to static IP configuration |

## Activating LACP Bond Interface

| Command | Description |
|---------|-------------|
| `nmcli con up bond0` | Activate bond interface |
| `nmcli con up bond-slave-eth0` | Activate first slave |
| `nmcli con up bond-slave-eth1` | Activate second slave |

## Verifying LACP Bond Configuration

| Command | Description |
|---------|-------------|
| `cat /proc/net/bonding/bond0` | View bond interface details |
| `nmcli con show bond0` | Show bond connection details |
| `nmcli device status` | Verify bond and slave status |

## Complete LACP Setup Script

```bash
# Create LACP bond interface
nmcli con add type bond con-name bond0 ifname bond0 bond.options "mode=802.3ad,miimon=100,downdelay=200,updelay=200,lacp_rate=1,xmit_hash_policy=layer3+4"

# Add slave interfaces
nmcli con add type bond-slave ifname eth0 master bond0
nmcli con add type bond-slave ifname eth1 master bond0

# Configure IP settings
nmcli con mod bond0 ipv4.addresses "192.168.1.100/24"
nmcli con mod bond0 ipv4.gateway "192.168.1.1"
nmcli con mod bond0 ipv4.method manual
nmcli con mod bond0 ipv4.dns "8.8.8.8,8.8.4.4"

# Activate all interfaces
nmcli con up bond0
nmcli con up bond-slave-eth0
nmcli con up bond-slave-eth1
```

## LACP Bond Options Explained

| Option | Description |
|--------|-------------|
| `mode=802.3ad` | IEEE 802.3ad Dynamic link aggregation (LACP) |
| `miimon=100` | Link monitoring frequency in milliseconds |
| `downdelay=200` | Wait 200 milliseconds before disabling a slave |
| `updelay=200` | Wait 200 milliseconds before enabling a slave |
| `lacp_rate=1` | LACP fast rate (1=fast, 0=slow) |
| `xmit_hash_policy=layer3+4` | Load balancing based on IP and Port |

# Port Channel (Bond) Configuration in Active-Backup Mode

## Creating Port Channel/Bond Interface

| Command | Description |
|---------|-------------|
| `nmcli con add type bond con-name bond0 ifname bond0 bond.options "mode=active-backup,miimon=100,downdelay=2000,updelay=5000,num_grat_arp=100"` | Create bond interface with specified parameters | 

## Adding Slave Interfaces to Bond

| Command | Description |
|---------|-------------|
| `nmcli con add type bond-slave ifname eth0 master bond0` | Add eth0 as slave to bond0 |
| `nmcli con add type bond-slave ifname eth1 master bond0` | Add eth1 as slave to bond0 |

## Configuring IP for Bond Interface

| Command | Description | 
|---------|-------------|
| `nmcli con mod bond0 ipv4.addresses "192.168.1.100/24"` | Set static IP for bond0 | 
| `nmcli con mod bond0 ipv4.gateway "192.168.1.1"` | Set gateway for bond0 | 
| `nmcli con mod bond0 ipv4.method manual` | Set to static IP configuration | 

# Activating Bond Interface

| Command | Description | 
|---------|-------------|
| `nmcli con up bond0` | Activate bond interface         | 
| `nmcli con up bond-slave-eth0` | Activate first slave  | 
| `nmcli con up bond-slave-eth1` | Activate second slave | 

## Verifying Bond Configuration

| Command | Description |
|---------|-------------|
| `cat /proc/net/bonding/bond0` | View bond interface details 
| `nmcli con show bond0` | Show bond connection details       
| `nmcli device status` | Verify bond and slave status         

## Complete Setup Example

```bash
# Create bond interface
nmcli con add type bond con-name bond0 ifname bond0 bond.options "mode=active-backup,miimon=100,downdelay=2000,updelay=5000,num_grat_arp=100"

# Add slave interfaces
nmcli con add type bond-slave ifname eth0 master bond0
nmcli con add type bond-slave ifname eth1 master bond0

# Configure IP settings
nmcli con mod bond0 ipv4.addresses "192.168.1.100/24"
nmcli con mod bond0 ipv4.gateway "192.168.1.1"
nmcli con mod bond0 ipv4.method manual
nmcli con mod bond0 ipv4.dns "8.8.8.8,8.8.4.4"

# Activate all interfaces
nmcli con up bond0
nmcli con up bond-slave-eth0
nmcli con up bond-slave-eth1

# Verify configuration
cat /proc/net/bonding/bond0
nmcli device status
```

## Bond Options Explained

| Option | Description |
|--------|-------------|
| `mode=active-backup` | One interface active, other in standby |
| `miimon=100` | Link monitoring frequency in milliseconds |
| `downdelay=2000` | Wait 2 seconds before disabling a slave |
| `updelay=5000` | Wait 5 seconds before enabling a slave |
| `num_grat_arp=100` | Number of gratuitous ARP messages to send |

