# Installation and VM Setup

## Virtual Machine

T-Pot is installed in a dedicated Debian virtual machine hosted on Proxmox VE.

| Setting | Value |
|---|---|
| Guest OS | Debian 13.7 Trixie |
| T-Pot | 24.04.1 |
| vCPU | 4 |
| RAM | 16 GiB |
| Disk | approximately 256 GiB |
| Swap | 8 GiB |
| NIC | VirtIO |
| Proxmox bridge | `vmbrlab` |
| Address | `<honeypot-address-cidr>` |
| Gateway | `<lab-gateway>` |
| DNS | `1.1.1.1` |

## Initial Network Validation

Useful checks after configuring the VM:

```bash
ip -br addr
ip route
ping 1.1.1.1
```

The VM should use `<lab-gateway>` as its default gateway.

## Disk Expansion

The virtual disk was expanded after VM creation.

The Debian partition and ext4 filesystem were then extended with tools such as:

```bash
sudo apt install cloud-guest-utils
sudo growpart /dev/sda 1
sudo resize2fs /dev/sda1
```

Always verify the actual disk and partition names before running filesystem commands.

## Swap

An 8 GiB swap file was created:

```bash
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Swap provides additional protection against memory pressure from Elasticsearch, Kibana, Logstash, Suricata, and the honeypot containers.

## T-Pot Installation

T-Pot was installed using its official installation workflow from a normal administrative user with `sudo` privileges.

After installation, T-Pot moves administrative services away from common honeypot ports.

Example management connection:

```bash
ssh -p 64295 <user>@<honeypot-ip>
```

TCP/22 is therefore available for Cowrie rather than the Debian SSH daemon.

## Service Validation

Useful checks after installation:

```bash
sudo docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
```

Core components include honeypots such as Cowrie, Dionaea, SNARE/Tanner, Heralding, Mailoney, SentryPeer and monitoring components such as Suricata, Elasticsearch, Logstash and Kibana.

## Next Steps

After the VM and T-Pot installation are working:

1. verify Internet access from the VM,
2. verify trusted-network isolation,
3. configure restricted egress,
4. expose selected honeypot services with DNAT,
5. verify packet counters and logs,
6. make tested rules persistent,
7. reboot-test the complete environment.

See [Network Configuration](network-configuration.md) for the traffic path and [Implementation Notes](implementation-notes.md) for the original detailed command history.
