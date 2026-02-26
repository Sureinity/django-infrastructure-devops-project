# Install Proxmox VE 9.1-1 on the Lab Node

## Goal

Install Proxmox VE on the node using the Proxmox VE `9.1-1` ISO and apply the initial installer disk settings for the lab baseline.

## Requirements

- Physical or virtual server with three available disks.
- Proxmox VE installer ISO: `https://enterprise.proxmox.com/iso/proxmox-ve_9.1-1.iso`.
- Console access to the target node.
- Disk layout target defined in `docs/reference/proxmox-node-disk-layout.md`.

## Steps

1. Download the Proxmox VE `9.1-1` ISO.
2. Boot the node from the ISO media.
3. Start the Proxmox VE installer.
4. Select `disk0` as the installation target.
5. Set the installer filesystem to `ext4`.
6. Set swap size to `4 GiB`.
7. Set installation size to `22 GiB` for the OS baseline.
8. Complete the installer prompts for region, credentials, and management network.
9. Finish installation and reboot into Proxmox VE.

## Verification

- Confirm Proxmox VE boots successfully and the web UI is reachable.
- Confirm the install disk contains `pve-root`, `pve-swap`, and installer-created `pve-data` thin storage.
- Confirm additional node disks remain available for `vmdata` and `backup-local` provisioning.

## Troubleshooting (Optional)

| Symptom | Cause | Action |
| --- | --- | --- |
| Installer does not detect all disks | Controller mode or disk attachment issue | Verify BIOS/UEFI storage controller configuration and rescan disks |
| Wrong disk selected during install | Install target selection error | Reinstall and select the correct OS target disk before proceeding |
| Post-install storage does not match expected baseline | Installer options differ from baseline | Recheck installer disk options and align with reference layout |
