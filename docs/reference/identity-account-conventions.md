# Identity and Account Conventions

## Metadata

| Field | Value |
| --- | --- |
| Component | Identity and access baseline |
| Version | v1 |
| Status | Active |
| Last reviewed | 2026-02-28 |

## Account Class Matrix

| Class | Naming Pattern | Linux Login | Proxmox Usage | MFA Requirement | Ownership Model |
| --- | --- | --- | --- | --- | --- |
| Human operator | `op_<name>` | Interactive login allowed by SSH key only | `op_<name>@pve` user for UI/API | Required | One account per person |
| Automation | `auto_<tool>` | Non-interactive automation login by SSH key | Scoped API token tied to automation user | Not applicable to API token | One account per automation system |
| Service | `svc_<service>` | Login disabled (`/usr/sbin/nologin`) | Not used for Proxmox login | Not applicable | Account mapped to owning service/team |
| Break-glass | `bg_admin` | Disabled by default; temporary enable only | Optional emergency Proxmox user only if approved | Required when enabled for human use | Explicitly approved emergency path |

## Naming Specification

| Object | Pattern | Example | Uniqueness Scope |
| --- | --- | --- | --- |
| Linux human user | `^op_[a-z0-9][a-z0-9_-]{1,30}$` | `op_ghlen` | Unique per organization |
| Linux automation user | `^auto_[a-z0-9][a-z0-9_-]{1,30}$` | `auto_ansible` | Unique per platform |
| Linux service user | `^svc_[a-z0-9][a-z0-9_-]{1,30}$` | `svc_django` | Unique per host/service scope |
| Linux break-glass user | `^bg_admin$` | `bg_admin` | Exactly one canonical name |
| Proxmox human principal | `op_<name>@pve` | `op_ghlen@pve` | Unique per Proxmox cluster |
| Proxmox API token | `auto_<tool>@pve!<token_id>` | `auto_terraform@pve!iac` | Unique per token namespace |

## Access Control Rules

| Control | Requirement | Scope |
| --- | --- | --- |
| Shared human accounts | Prohibited. Human access must map 1:1 to an individual identity. | Linux hosts and Proxmox users |
| Root account usage | `root@pam` is emergency-only and not used for routine administration. | Proxmox node and cluster access |
| SSH authentication | Key-based login only. Password authentication is disabled. | Managed Linux hosts |
| Proxmox human MFA | Required for all human admin users in `pve` realm. | Proxmox UI/API for human users |
| Service login shell | Service users must use `/usr/sbin/nologin`. | Managed Linux hosts |
| Sudo delegation | Sudo access is role/profile based with explicit allowlists. | Managed Linux hosts |
| API token scope | Tokens must be least-privilege and task-scoped. | Terraform/automation access to Proxmox API |

## Identity Schema Keys

| Key | Type | Required | Description |
| --- | --- | --- | --- |
| `identity_human_users` | list(object) | Yes | Human operator identities and access attributes |
| `identity_service_users` | list(object) | Yes | Non-login service account definitions |
| `identity_automation_user` | object | Yes | Automation identity used by Ansible/Terraform |
| `identity_sudo_profiles` | map(list(string)) | Yes | Named sudo command allowlists by profile |
| `identity_ssh_policy` | object | Yes | SSH daemon and login policy baseline |
| `identity_rotation_policy` | object | Yes | Credential and access review cadences |

## Schema Field Contract

| Schema Key | Required Fields |
| --- | --- |
| `identity_human_users[]` | `username`, `full_name`, `linux_groups`, `ssh_public_keys`, `proxmox_username`, `mfa_required`, `state` |
| `identity_service_users[]` | `username`, `owner_team`, `host_scope`, `shell`, `state` |
| `identity_automation_user` | `username`, `ssh_public_keys`, `sudo_profile`, `proxmox_username`, `api_token_id`, `state` |
| `identity_sudo_profiles` | `profile_name -> [allowed_command, ...]` |
| `identity_ssh_policy` | `permit_root_login`, `password_authentication`, `allow_groups` |
| `identity_rotation_policy` | `ssh_key_review_days`, `api_token_review_days`, `access_review_days` |

## Policy Defaults

| Setting | Default |
| --- | --- |
| `identity_ssh_policy.permit_root_login` | `no` |
| `identity_ssh_policy.password_authentication` | `no` |
| `identity_rotation_policy.ssh_key_review_days` | `90` |
| `identity_rotation_policy.api_token_review_days` | `180` |
| `identity_rotation_policy.access_review_days` | `30` |

## Allowed State Values

| Field | Allowed Values |
| --- | --- |
| `identity_human_users[].state` | `active`, `disabled`, `deprovisioned` |
| `identity_service_users[].state` | `active`, `disabled` |
| `identity_automation_user.state` | `active`, `disabled` |
