# DirtyFrag Mitigation Playbook

Ansible playbook to safely apply the kernel module blacklist workaround for the **DirtyFrag** privilege escalation vulnerability, which chains the `xfrm-ESP` and `RxRPC` page-cache write bugs to gain root on unpatched Linux systems.

## What it does

For each host it:
1. Checks whether `esp4`, `esp6`, or `rxrpc` are currently loaded or built into the kernel
2. If safe, writes `/etc/modprobe.d/dirtyfrag.conf` blocking all three modules from loading
3. If not safe (e.g. host uses an IPsec VPN), skips and reports why

## Requirements

- Ansible 2.9+
- `become: true` (sudo/root access on targets)

## Usage

```bash
# Dry run first
ansible-playbook -i inventory dirtyfrag-fix.yml --check

# Apply
ansible-playbook -i inventory dirtyfrag-fix.yml
```

## When a host is skipped

A host is skipped if any of the three modules are **loaded** (actively in use, e.g. an IPsec VPN) or **built-in** to the kernel (`=y` in kernel config, meaning modprobe blacklisting has no effect). The play will tell you which modules triggered the skip — those hosts need a proper kernel patch or manual review.

## Caveats

- Blacklisting `esp4`/`esp6` breaks IPsec (IKEv2, L2TP/IPsec). OpenVPN and WireGuard are unaffected.
- The definitive fix is a kernel update from your distribution. This is a stopgap.
- If a module is built-in (`=y`), neither this playbook nor any modprobe config can block it.
