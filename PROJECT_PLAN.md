# Project Plan

This file tracks execution status for the lab implementation.

## Goal

Build a production-structured Proxmox lab with `staging` and `prod` environments, provisioned by Terraform and configured by Ansible.

## Scope

In scope:
- Proxmox node baseline and storage architecture
- Single Proxmox node on a server target (bare metal baseline; VM-hosted deployment is allowed)
- Terraform provisioning for `staging` and `prod`
- Ansible-based server configuration and security baseline
- Documentation in Diataxis format

Out of scope:
- Multi-node Proxmox HA cluster implementation
- Nested-virtualization-specific tuning beyond baseline bare-metal assumptions
- External identity provider integration
- Full production SRE runbook depth in root README

## Current Phase

- Current phase: Node baseline and infrastructure foundation.
- Priority environment: `staging`.

## Milestones

- Foundation: node baseline, storage policy, repo structure, core docs
- Staging: template, provisioning, configuration, end-to-end validation
- Production: mirror and promote validated staging patterns

## Task Tracking Rules

| Rule | Standard |
| --- | --- |
| Task size | Each sub-task must fit in 30-90 minutes |
| Structure | Use 3 levels: workstream -> parent task -> sub-task |
| Status model | Use `[ ]` and `[x]`; use `[BLOCKED: reason]` when execution is blocked |
| Completion criteria | Every parent task and sub-task includes `(Done when: ...)` |
| Recurring work | Track cadence in `Recurring Operations` |

## Checklist

### Foundation

- [x] Stabilize repository and documentation baseline (Done when: core repository structure and Diataxis indexes are present)
  - [x] Create Terraform, Ansible, and docs directory structure. (Done when: top-level repo paths match planned layout)
  - [x] Establish Diataxis quadrants and starter templates. (Done when: `tutorials`, `how-to-guides`, `reference`, and `explanation` each contain a `TEMPLATE.md`)
  - [x] Publish Proxmox node disk layout reference for 90 GiB. (Done when: `docs/reference/proxmox-node-disk-layout.md` exists and is indexed)
  - [x] Publish initial Proxmox installation how-to guide. (Done when: `docs/how-to-guides/install-proxmox-ve-node.md` exists and is indexed)

### Proxmox Node Baseline

- [ ] Provision `vmdata` storage on `sdb` (Done when: `pvesm status` reports active `vmdata` as `lvmthin`)
  - [ ] Create the `LVM-thin` pool for VM disk data on `sdb`. (Done when: thin pool exists and is visible via `lvs`)
  - [ ] Register `vmdata` in Proxmox storage config with `images,rootdir`. (Done when: storage content types match target policy)

- [ ] Provision `backup-local` storage on `sdc` (Done when: `pvesm status` reports active directory storage `backup-local`)
  - [ ] Format and mount `sdc` with `ext4` for backup staging. (Done when: mount is persistent and visible in `findmnt`)
  - [ ] Register `backup-local` in Proxmox storage config with `backup` content. (Done when: `backup-local` accepts backup jobs)

- [ ] Enforce node storage placement policy (Done when: VM disks are constrained to intended storage backends)
  - [ ] Restrict `local` storage content to `iso,vztmpl,snippets`. (Done when: `local` no longer allows VM disk content)
  - [ ] Confirm VM disk placement defaults to `vmdata` for new VMs. (Done when: test VM disk is created on `vmdata`)

- [ ] Configure Proxmox node cloud-init prerequisites (Done when: node-level cloud-init storage and metadata workflows are validated)
  - [ ] Verify snippet-capable storage path for cloud-init custom data on node. (Done when: snippet file can be created and referenced from Proxmox storage config)
  - [ ] Validate cloud-init metadata rendering for template VM IDs. (Done when: `qm cloudinit dump` returns expected `user`, `network`, and `meta` content)
  - [ ] Validate first-boot cloud-init application on a test clone. (Done when: cloned VM applies hostname, user, SSH key, and network config at first boot)

- [ ] Enable backup workflow and off-node copy strategy (Done when: local backups are scheduled and off-node destination is defined)
  - [ ] Create nightly local backup schedule to `backup-local`. (Done when: backup job exists and next run is scheduled)
  - [ ] Define off-node destination policy and transfer path. (Done when: documented destination and transfer method exist)

### Security and Identity

- [ ] Implement Linux identity baseline for operators and automation (Done when: named operator model and `ansible` automation user are codified)
  - [ ] Define user and group schema in Ansible inventory variables. (Done when: schema keys exist in env vars and pass lint)
  - [ ] Define secret-bearing identity data in Ansible Vault placeholders. (Done when: required vault variables are created without plaintext secrets)

- [ ] Implement non-login service account model (Done when: service users are codified with non-interactive shell policy)
  - [ ] Define `svc_*` account policy and ownership mapping in inventory schema. (Done when: service user definitions include role ownership and host scope)
  - [ ] Enforce `/usr/sbin/nologin` shell for service users on managed hosts. (Done when: service account shell audit confirms non-login shells)

- [ ] Enforce SSH hardening policy on managed hosts (Done when: root and password SSH authentication are disabled)
  - [ ] Set `PermitRootLogin no` and `PasswordAuthentication no` in managed SSH config. (Done when: config audit confirms both values)
  - [ ] Validate key-based login for expected admin and automation users. (Done when: login succeeds with key and fails with password)

- [ ] Implement least-privilege sudo profiles (Done when: allowed commands are profile-scoped and verified)
  - [ ] Create sudo allowlist profiles for admin and automation roles. (Done when: managed sudoers files exist and validate with `visudo -cf`)
  - [ ] Test denial of unauthorized sudo commands. (Done when: blocked command attempts are denied as expected)

- [ ] Configure Proxmox platform access baseline (Done when: `pve` realm admins use MFA and automation uses scoped API token)
  - [ ] Create or verify admin users in `pve` realm with MFA enabled. (Done when: human admin login requires MFA)
  - [ ] Create scoped API token for Terraform automation. (Done when: token can run required Terraform actions only)
  - [ ] Validate token ACL boundaries and denied actions. (Done when: unauthorized API operations fail with permission errors)

- [ ] Implement break-glass access policy (Done when: emergency path is disabled by default and temporary enable workflow is documented)
  - [ ] Document temporary break-glass enable/disable procedure with approval requirement. (Done when: procedure exists in docs with explicit expiry step)
  - [ ] Validate disabled-default state in host and Proxmox access settings. (Done when: no permanent break-glass account is active in steady state)

- [ ] Implement identity lifecycle controls (Done when: onboarding, offboarding, and key revocation workflows are tracked and testable)
  - [ ] Create onboarding task flow for operator account provisioning. (Done when: new operator can be onboarded through inventory + playbook execution path)
  - [ ] Create offboarding task flow for account disable and key revocation. (Done when: deprovisioned user access is denied on validation check)

- [ ] Publish identity schema contract and audit outputs (Done when: schema keys and periodic audit artifacts are defined and linked)
  - [ ] Publish reference schema keys for identity objects (`identity_human_users`, `identity_service_users`, `identity_automation_user`, `identity_sudo_profiles`, `identity_ssh_policy`, `identity_rotation_policy`). (Done when: schema reference is available under `docs/reference/`)
  - [ ] Define monthly identity audit output format for active users, keys, sudo profiles, and drift notes. (Done when: template/report format is documented and linked from plan)

### Staging Environment

- [ ] Build Debian cloud-init golden template (Done when: template can clone and bootstrap via cloud-init)
  - [ ] Create base Debian VM with `qemu-guest-agent` and cloud-init support. (Done when: cloned VM receives user/network config at first boot)
  - [ ] Convert base VM to reusable template and verify clone workflow. (Done when: one successful test clone passes SSH readiness check)

- [ ] Implement Terraform staging stack entrypoints (Done when: staging edge/app stacks can plan cleanly)
  - [ ] Create stack skeletons for `staging/edge` and `staging/app` with separate state config. (Done when: `terraform init` and `terraform validate` pass per stack)
  - [ ] Wire module inputs for compute, storage, and network per staging sizing. (Done when: `terraform plan` resolves expected resources)

- [ ] Implement Ansible staging inventory and role mapping (Done when: staging inventory can execute baseline and service playbooks)
  - [ ] Populate staging hosts and group variables for edge and app roles. (Done when: inventory graph renders expected groups)
  - [ ] Execute baseline playbook in check mode against staging targets. (Done when: no fatal errors in check run)

- [ ] Validate staging end-to-end deployment path (Done when: edge-to-app path and management access both work)
  - [ ] Verify service routing path from edge to app using defined health endpoint. (Done when: health endpoint returns success through edge)
  - [ ] Verify admin access path and host hardening controls. (Done when: admin login works via approved path and blocked methods fail)

### Production Environment

- [ ] Mirror Terraform structure for production stacks (Done when: `prod/edge`, `prod/app`, `prod/observability` entrypoints are aligned to staging pattern)
  - [ ] Create production stack skeletons with independent backend config. (Done when: each stack initializes without staging state reuse)
  - [ ] Apply production-specific sizing and policy variables. (Done when: production plans show expected deltas only)

- [ ] Mirror Ansible production inventory and policies (Done when: production inventory matches structure with environment-specific values)
  - [ ] Populate production hosts/group_vars/host_vars from validated staging pattern. (Done when: inventory lint/check passes)
  - [ ] Verify production-only secrets and identity variables are set via vault. (Done when: no production secrets are in plaintext files)

- [ ] Promote validated staging patterns to production execution (Done when: production rollout checklist is complete and verified)
  - [ ] Execute production dry-run validation for Terraform and Ansible. (Done when: no critical blockers remain)
  - [ ] Execute controlled production rollout and post-deploy verification. (Done when: post-deploy checks pass and rollback path is documented)

## Recurring Operations

| Task | Cadence | Last Completed | Next Due | Owner |
| --- | --- | --- | --- | --- |
| SSH key rotation review | Every 90 days | - | - | Platform |
| Proxmox API token rotation review | Every 180 days | - | - | Platform |
| Backup restore drill | Every 30 days | - | - | Platform |
| Access and sudo policy review | Every 30 days | - | - | Platform |
| Identity onboarding/offboarding audit | Every 30 days | - | - | Platform |

## Risks and Blockers

- Risk: single-node architecture remains a single point of failure.
- Risk: limited disk and RAM capacity increase contention risk during concurrent workloads.
- Blockers: none active. Use `[BLOCKED: reason]` on affected checklist items.

## Decisions Log

- 2026-02-27: Use `PROJECT_PLAN.md` as the canonical execution tracking document.
- 2026-02-27: Keep Diataxis docs intent-pure and link task details from this plan.
- 2026-02-27: Standardize task granularity to 30-90 minute sub-tasks with inline completion criteria.

## Next Actions

- [ ] Verify current node storage state before provisioning `vmdata`. (Done when: `lsblk`, `vgs`, and `pvesm status` outputs are captured in session notes)
- [ ] Create `vmdata` storage and validate visibility in Proxmox. (Done when: `vmdata` appears as active `lvmthin` in `pvesm status`)
- [ ] Create `backup-local` storage and validate backup content type. (Done when: `backup-local` appears active and backup job target selection works)

## Notes

- Update this plan as tasks start or complete.
- Keep detailed procedures in quadrant-specific docs and link from this file.
