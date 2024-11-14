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
| `nmcli connection add type ethernet con-name "eth0" ifname eth0` | Create basic ethernet connection |
| `nmcli con add type ethernet con-name "eth0" ifname eth0 ip4 192.168.1.100/24 gw4 192.168.1.1` | Create connection with static IP |
| `nmcli con add type ethernet con-name "eth0" ifname eth0 ipv4.method auto` | Create connection with DHCP |
| `nmcli con add type ethernet con-name "eth0" ifname eth0 ip4 192.168.1.100/24 gw4 192.168.1.1 ip4.dns "8.8.8.8,8.8.4.4"` | Create connection with static IP, gateway and DNS |

## Modifying Existing Connections

| Command | Description |
|---------|-------------|
| `nmcli con mod "eth0" ipv4.addresses "192.168.1.100/24"` | Change IP address |
| `nmcli con mod "eth0" ipv4.gateway "192.168.1.1"` | Change gateway |
| `nmcli con mod "eth0" ipv4.dns "8.8.8.8,8.8.4.4"` | Change DNS servers |
| `nmcli con mod "eth0" ipv4.method manual` | Change to static IP configuration |
| `nmcli con mod "eth0" ipv4.method auto` | Change to DHCP configuration |
| `nmcli con mod "eth0" connection.autoconnect yes` | Enable auto-connect |
| `nmcli con mod "eth0" connection.autoconnect no` | Disable auto-connect |

## Managing Connection State

| Command | Description |
|---------|-------------|
| `nmcli con up "eth0"` | Activate a connection |
| `nmcli con down "eth0"` | Deactivate a connection |
| `nmcli con reload` | Reload all connection files |
| `nmcli device reapply eth0` | Reapply connection settings |
| `nmcli device disconnect eth0` | Disconnect interface |
| `nmcli device connect eth0` | Connect interface |

## Advanced Configuration

| Command | Description |
|---------|-------------|
| `nmcli con mod "eth0" 802-3-ethernet.mtu 9000` | Change MTU size |
| `nmcli con mod "eth0" ethernet.mac-address AA:BB:CC:DD:EE:FF` | Set MAC address |
| `nmcli con mod "eth0" ipv4.never-default true` | Prevent as default route |
| `nmcli con mod "eth0" connection.interface-name eth0` | Change interface name |
| `nmcli con mod "eth0" ipv4.dns-search "example.com"` | Set DNS search domain |

## Deleting Connections

| Command | Description |
|---------|-------------|
| `nmcli con delete "eth0"` | Delete a connection profile |
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
