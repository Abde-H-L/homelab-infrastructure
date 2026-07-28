# Phase 8 — Backups (srv-backup)

## What was done

Updated the existing `ansible/playbooks/docker.yml` playbook to also install Docker on **srv-backup**.

Deployed **Duplicati** using `docker/backup/docker-compose.yml` and configured two backup jobs:

- **postgres-dump**: daily backups of the PostgreSQL database from **srv-db**.
- **docker-web-volumes**: backups of the n8n Docker volume from **srv-web**.

Both backup jobs use a retention policy of **7 daily** and **4 weekly** backups.

To make the data available to Duplicati, both remote directories were mounted on **srv-backup** using **SSHFS**, with the mounts configured in `/etc/fstab` using the `_netdev` option so they are restored automatically after boot.

## Decisions

- Reused the existing `docker.yml` playbook again instead of creating a separate one for **srv-backup**.
- Chose **SSHFS** instead of NFS because it takes advantage of the SSH keys already configured in Phase 4 without introducing another service into the lab.
- Used a scheduled `pg_dumpall` script on **srv-db** instead of backing up PostgreSQL's data directory directly. SQL dumps are portable and can be safely generated while the database is running.
- Configured a retention policy of **7 daily** and **4 weekly** backups, which is enough for a homelab without using too much storage.

## Problems encountered

- **SSHFS couldn't mount the Docker volume from srv-web** because the Docker volume directory is only accessible by `root`. Instead of changing ownership of Docker's files, I used `setfacl` to grant the required permissions so the management user could access the directory without affecting Docker itself.
- **The SSHFS mounts worked manually but not after reboot.** The mounts are created by systemd as `root`, and the root user had never accepted the SSH fingerprints of **srv-db** and **srv-web**. After connecting once as `root` and accepting both host keys, the automatic mounts worked correctly after reboot.

## Verification

Ran both Duplicati jobs manually and confirmed they completed successfully without errors.

Also verified that the SSHFS mounts were restored automatically after reboot by checking the mounted filesystems with:

```bash
df -h
```

Both backup sources were available, and Duplicati reported a successful backup for each job.