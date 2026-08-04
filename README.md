# Multi-Site Enterprise Network Lab

A 14-VM, 3-site enterprise network built in VMware Workstation for the Network
Administration AEC at John Abbott College. Domain: `arasaka.local`

## Topology

Three geographic sites plus a remote VPN client.

| Site | Components |
|------|-----------|
| Site 1 (HQ) | pfSense, Cisco IOS CSR router, primary DC, web server, file/backup server |
| Site 2 | pfSense, additional DC, Linux file server, AlmaLinux client |
| Site 3 (Branch) | pfSense, RODC, Windows client |
| Remote | OpenVPN client |

**Platforms:** Windows Server 2022/2025, AlmaLinux 9.8, Windows 11, pfSense, Cisco IOS

## What was built

### Networking
- 3 site-to-site IPSec tunnels (IKEv2, PSK, AES-256, SHA-256, DH Group 14)
- Remote-access OpenVPN with a certificate authority
- NAT port forwarding (WAN:443 → internal web server), DMZ segment
- VLANs, inter-VLAN routing, DHCP relay

### Active Directory
- Forest with an additional domain controller and a read-only DC at the branch site
- AD Sites and Services with subnet-to-site mapping to optimize authentication traffic
- GPO drive mappings using item-level targeting by security group
- Departmental file shares with NTFS and share-level permissions
- CIFS/Kerberos auto-mount for Linux clients

### Infrastructure
- RAID 1 (Windows mirrored volume, Linux /home) and RAID 5 (backup storage)
- Windows Server Backup for shares and DC system state
- rsync + cron backups for Linux /home and /var/www
- Restore testing performed and documented for both platforms
- Apache with SSL, certificates issued from an internal Root CA and distributed domain-wide
- Zabbix 7.4 monitoring with a MariaDB backend, 4 hosts

## Troubleshooting notes

Write-ups of problems encountered and how they were diagnosed:

- [Subnet failure caused by hypervisor NIC mapping mismatch](docs/nic-mapping-failure.md)
- [CIFS/Kerberos mount failure — cifs-utils 7.6 cruid regression](docs/cifs-kerberos-cruid.md)
- [pfSense IPSec self-originated ping behaviour](docs/pfsense-ipsec-ping.md)
