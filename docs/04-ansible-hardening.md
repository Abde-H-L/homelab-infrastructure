# Phase 4 — Ansible from srv-ops

## What was done

Installed Ansible on **srv-ops** and generated an SSH key pair. Then copied the public key to every node using `ssh-copy-id` so Ansible could connect without asking for passwords.

Created the `ansible/inventory.ini` file with all 7 nodes grouped by their role (`gw`, `ops`, `web`, `db`, `monitor` and `backup`) and wrote a hardening playbook to update packages, enable UFW, install Fail2Ban and configure unattended upgrades.

Finally, ran the playbook against the whole inventory:

```bash
ansible-playbook -i inventory.ini playbooks/hardening.yml
```

## Decisions

- Used **srv-ops** as the Ansible control node instead of the host machine to keep everything inside the lab.
- Defined `ansible_user` and `ansible_ssh_private_key_file` under `[all:vars]` since every machine uses the same management user.
- Switched from `es.archive.ubuntu.com` to `archive.ubuntu.com` after repeated connection problems with the Spanish mirror.

## Problems encountered

- **SSH host key verification failed** on `srv-gw` and `srv-ops` because I had never connected to them manually before. Fixed it by connecting once over SSH and accepting the fingerprint.
- **Permission denied (publickey,password)** on the same two machines because I forgot to run `ssh-copy-id`. After copying the SSH key, Ansible connected correctly.
- **sudo: interactive authentication is required** because the playbook uses `become: true` and sudo expected a password. I fixed it by creating a `NOPASSWD:ALL` rule for the management user inside `/etc/sudoers.d/`.
- **dpkg lock** errors while updating packages because `unattended-upgrades` was already running after boot. Waiting a few seconds and running the playbook again solved the issue.
- **IPv6 connection errors** on `srv-ops`. The node was trying to reach the Ubuntu mirror through IPv6 even though the lab only had IPv4 NAT. Disabled IPv6 using `sysctl`.
- **Unstable apt mirror.** `es.archive.ubuntu.com` kept timing out, so I switched the affected machines to `archive.ubuntu.com`.
- **DNS stopped working** on `srv-web`, `srv-monitor` and `srv-backup` even though they could still ping the gateway. After checking with `tcpdump`, `nft` and `ufw`, I found that enabling UFW on **srv-gw** changed the kernel's forward policy to **DROP**, overriding my nftables forwarding rule. ICMP traffic still worked, but new TCP/UDP connections (including DNS) were blocked. Fixed it by allowing forwarded traffic explicitly:

```bash
ufw route allow in on enp0s8 out on enp0s3
```

instead of allowing all forwarded traffic.

## Verification

The playbook finished successfully on all seven machines:

```text
PLAY RECAP
...
failed=0
```

Also checked that `ufw` and `fail2ban` were running correctly on every node, and verified that DNS resolution and internet access were working again from `srv-web` after fixing the forwarding issue.