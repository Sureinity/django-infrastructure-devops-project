# Configure Proxmox and Ceph APT Repositories for No-Subscription

## Goal

Switch a Proxmox VE node from enterprise APT repositories to no-subscription repositories, including Ceph package sources when Ceph packages are required.

## Requirements

- Direct console or SSH access to the Proxmox node.
- Root or sudo access.
- Outbound internet connectivity from the node.
- Proxmox VE `9.x` node baseline (Debian `trixie`).
- Official repository format reference: `https://pve.proxmox.com/wiki/Package_Repositories`.

## Steps

1. Back up current APT repository files.

```bash
cp -a /etc/apt/sources.list /etc/apt/sources.list.bak.$(date +%F-%H%M%S) 2>/dev/null || true
cp -a /etc/apt/sources.list.d /etc/apt/sources.list.d.bak.$(date +%F-%H%M%S)
```

2. Disable enterprise repository entries when present.

```bash
TS=$(date +%F-%H%M%S)

if [ -f /etc/apt/sources.list.d/pve-enterprise.list ]; then
  mv /etc/apt/sources.list.d/pve-enterprise.list /etc/apt/sources.list.d/pve-enterprise.list.disabled.${TS}
fi

if [ -f /etc/apt/sources.list.d/pve-enterprise.sources ]; then
  mv /etc/apt/sources.list.d/pve-enterprise.sources /etc/apt/sources.list.d/pve-enterprise.sources.disabled.${TS}
fi

if [ -f /etc/apt/sources.list.d/ceph.sources ]; then
  mv /etc/apt/sources.list.d/ceph.sources /etc/apt/sources.list.d/ceph.sources.disabled.${TS}
fi

if [ -f /etc/apt/sources.list.d/ceph.list ]; then
  mv /etc/apt/sources.list.d/ceph.list /etc/apt/sources.list.d/ceph.list.disabled.${TS}
fi
```

3. Configure Proxmox no-subscription repository using the official `deb822` format.

```bash
cat >/etc/apt/sources.list.d/proxmox.sources <<'EOF'
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
```

4. If Ceph packages are required on this node, configure the Ceph no-subscription repository in `deb822` format.

```bash
cat >/etc/apt/sources.list.d/ceph.sources <<'EOF'
Types: deb
URIs: http://download.proxmox.com/debian/ceph-squid
Suites: trixie
Components: no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
```

5. Refresh package metadata.

```bash
apt update
```

## Verification

- Confirm active repository files contain no enterprise entry.

```bash
find /etc/apt/sources.list.d -maxdepth 1 -type f \( -name "*.list" -o -name "*.sources" \) -print0 | \
  xargs -0 grep -n "enterprise.proxmox.com" || true
grep -n "pve-no-subscription" /etc/apt/sources.list.d/proxmox.sources
grep -n "Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg" /etc/apt/sources.list.d/proxmox.sources
```

- Confirm `apt update` completes without enterprise authorization failures.
- If Ceph packages are required, confirm Ceph no-subscription repository values:

```bash
grep -n "ceph-squid" /etc/apt/sources.list.d/ceph.sources
grep -n "Components: no-subscription" /etc/apt/sources.list.d/ceph.sources
```

## Troubleshooting (Optional)

| Symptom | Cause | Action |
| --- | --- | --- |
| `apt update` still contacts `enterprise.proxmox.com` | Another active enterprise source file remains | Search `/etc/apt/sources.list*` and `/etc/apt/sources.list.d/*`, then disable remaining enterprise entries |
| `apt update` cannot resolve `download.proxmox.com` | DNS or outbound routing issue | Verify node DNS config, default route, and upstream firewall policy |
| `apt update` fails with Ceph package source errors | `ceph.sources` is missing, malformed, or mismatched to `trixie`/`ceph-squid` | Recreate `/etc/apt/sources.list.d/ceph.sources` with the official stanza and run `apt update` again |
| `apt update` reports missing key or signature validation failure | Proxmox archive keyring package is missing or incorrect | Verify `/usr/share/keyrings/proxmox-archive-keyring.gpg` exists and reinstall keyring package if needed |
