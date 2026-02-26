# Proxmox Node Disk Layout (Lab Target: 90 GiB)

## Capacity Profile

| Parameter | Value |
| --- | --- |
| Total node storage budget | 90 GiB |
| Hypervisor | Proxmox VE |
| Host filesystem strategy | `ext4` + `LVM-thin` |
| Lab objective | Production-style topology under constrained capacity |

## Logical Disk Allocation

| Logical Disk | Size | Type | Filesystem | Primary Use |
| --- | ---: | --- | --- | --- |
| `disk0-os` | 22 GiB | OS/system disk | `ext4` | Proxmox host OS, boot, swap |
| `disk1-vmdata` | 58 GiB | VM data disk | `LVM-thin` | VM system/data volumes |
| `disk2-backup` | 10 GiB | Backup staging disk | `ext4` | Local `vzdump` staging |

## Proxmox Storage Definitions

| Storage ID | Backing | Type | Content Types | Role |
| --- | --- | --- | --- | --- |
| `local` | `disk0-os` | Directory | `iso,vztmpl,snippets` | Installation assets and snippets |
| `vmdata` | `disk1-vmdata` | LVM-thin | `images,rootdir` | Primary VM disk placement |
| `backup-local` | `disk2-backup` | Directory | `backup` | Local backup retention window |

## Capacity Policy for `vmdata` (58 GiB)

| Allocation Domain | Reserved Capacity |
| --- | ---: |
| Production workloads | 34 GiB |
| Staging workloads | 15 GiB |
| Shared/ops utility space | 4 GiB |
| Free safety headroom | 5 GiB |

## VM Disk Budget Baseline

| VM | Disk Plan | Total |
| --- | --- | ---: |
| `prod-edge-01` | root `5 GiB` | 5 GiB |
| `prod-app-01` | root `7 GiB` + docker `10 GiB` + postgres `12 GiB` | 29 GiB |
| `staging-edge-01` | root `4 GiB` | 4 GiB |
| `staging-app-01` | root `5 GiB` + app/data `6 GiB` | 11 GiB |
| `ops-01` | root `4 GiB` | 4 GiB |

## Guest Filesystem Baseline

| Workload Disk Type | Filesystem | Mount Option Baseline |
| --- | --- | --- |
| Linux root disk | `ext4` | defaults |
| Docker data disk | `ext4` | defaults |
| PostgreSQL data disk | `ext4` | `noatime` |

## Current Post-Install Host State

| Device | State |
| --- | --- |
| `sda` | Proxmox-installed disk containing EFI + `pve` volume group |
| `pve-root` | Mounted at `/` (`ext4`) |
| `pve-swap` | Active swap volume |
| `pve-data` | Existing thin pool created by installer |
| `sdb` | Unallocated at install completion |
| `sdc` | Unallocated at install completion |

## Storage Operation Constraints

| Control | Target |
| --- | --- |
| Snapshot use | Short-lived rollback points only |
| Backup retention on node | 1-2 restore points |
| Backup destination policy | Off-node copy required for recovery |
| Utilization warning threshold | 70% |
| Utilization critical threshold | 85% |
