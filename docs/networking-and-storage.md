# Networking and storage

## Network design

The previous Proxmox host used Wi-Fi (`wlan0`) for its upstream connection.
Instead of attaching the VM directly to the wireless interface, it used an
internal Proxmox bridge named `vmbr1`.

| Network component | Previous value / purpose |
|---|---|
| Upstream interface | `wlan0` |
| Proxmox bridge | `vmbr1` |
| Bridge address | `10.10.100.1/24` |
| VM subnet | `10.10.100.0/24` |
| VM network model | VirtIO |

The bridge had no physical bridge ports (`bridge-ports none`). The host
provided routing and NAT for guests on the private VM subnet.

## NAT and forwarding

The previous `/etc/network/interfaces` configuration enabled IPv4 forwarding
when the bridge came up and added a source-NAT rule:

```bash
post-up echo 1 > /proc/sys/net/ipv4/ip_forward
post-up iptables -t nat -A POSTROUTING -s '10.10.100.0/24' -o wlan0 -j MASQUERADE
post-down iptables -t nat -D POSTROUTING -s '10.10.100.0/24' -o wlan0 -j MASQUERADE
```

In practical terms:

1. The Ubuntu VM sends traffic through `vmbr1`.
2. Proxmox routes the traffic out through `wlan0`.
3. The MASQUERADE rule rewrites the source address to the host's upstream
   address so replies can return through the host.

This design was used because the host's upstream connection was wireless.
A routed/NATed virtual network avoids relying on a conventional bridged
Ethernet connection to the Wi-Fi network.

**Redeployment checks:** confirm the actual wireless interface name, ensure
IPv4 forwarding persists as intended, verify firewall policy, and test both
DNS resolution and outbound connectivity from the guest. The interface name
and network details may change after reinstalling Proxmox.

## Proxmox storage

The previous Proxmox storage configuration defined:

- `local`: directory storage at `/var/lib/vz`, for ISO images, templates,
  backups, and imports.
- `local-lvm`: LVM-thin storage using the `data` thin pool in volume group
  `pve`, for VM disks and container root filesystems.

The Ubuntu VM used a 100 GB virtual disk on `local-lvm` and an Ubuntu Server
installer ISO on `local`.

## Backup and data separation

The previous migration backup included a VM backup and application data.
Those artifacts are intentionally excluded from this repository because they
are large and may contain private information.

Keep these categories separate:

- **Git repository:** text configuration, Compose definitions, documentation,
  and sanitized examples.
- **Backups:** VM images and database dumps, stored privately and tested for
  restore.
- **Application data:** photo libraries, uploads, and database files, stored
  on dedicated storage and backed up separately.
- **Secrets:** real environment files, API keys, passwords, certificates, and
  private keys, stored outside Git.

A backup is only useful if it can be restored. After redeployment, perform a
test restore and document the result without publishing the backup itself.
