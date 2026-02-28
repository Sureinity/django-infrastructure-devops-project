# Recover Proxmox Node Network After Bridge Port Mismatch

## Goal

Restore Proxmox node management connectivity when `vmbr0` is configured with a non-existent physical interface (for example `nic0` instead of `ens18`).

## Requirements

- Direct console access to the Proxmox node.
- Root or sudo access on the node.
- Correct management network values:
  - Node IP/CIDR
  - Gateway IP
  - Physical uplink interface name (for example `ens18`)

## Steps

1. Confirm the physical interface name.

```bash
ip -br link
ls -1 /sys/class/net
```

2. Back up the current network configuration.

```bash
cp /etc/network/interfaces /etc/network/interfaces.bak.$(date +%F-%H%M%S)
```

3. Open `/etc/network/interfaces` and replace the wrong bridge port with the real interface name.

```ini
auto lo
iface lo inet loopback

iface ens18 inet manual

auto vmbr0
iface vmbr0 inet static
    address 10.31.0.103/24
    gateway 10.31.0.1
    bridge-ports ens18
    bridge-stp off
    bridge-fd 0
```

4. Apply the configuration.

```bash
ifreload -a
```

5. If connectivity is still not restored, cycle the interfaces from console.

```bash
ifdown vmbr0 --force || true
ifdown ens18 --force || true
ifup ens18
ifup vmbr0
```

## Verification

- Confirm both interfaces are up.

```bash
ip -br link show ens18 vmbr0
ip -br addr show vmbr0
```

- Confirm routing is correct.

```bash
ip route
ping -c 3 10.31.0.1
```

- Confirm remote access works:
  - SSH to the node succeeds.
  - Proxmox web UI is reachable at `https://10.31.0.103:8006`.

## Troubleshooting (Optional)

| Symptom | Cause | Action |
| --- | --- | --- |
| `ifreload -a` reports unknown interface in `bridge-ports` | Bridge config references a NIC name that does not exist | Use `ip -br link` to identify the real NIC and update `bridge-ports` |
| `ens18` and `vmbr0` remain `DOWN` after reload | Link is not connected at lower layer (cable/vSwitch/port group/VLAN issue) | Validate physical or virtual switch connection and VLAN assignment, then re-run `ifreload -a` |
| Gateway ping fails but link is `UP` | Wrong IP, subnet, gateway, or upstream route | Re-verify address plan and upstream gateway settings |
