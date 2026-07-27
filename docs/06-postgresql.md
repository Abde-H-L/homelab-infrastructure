# Phase 6 — PostgreSQL High Availability (srv-db + srv-db2)

## What was done

Installed PostgreSQL 18 on **srv-db** and **srv-db2** using the `ansible/playbooks/postgresql.yml` playbook.

Configured **srv-db** as the primary server by enabling streaming replication, creating a dedicated replication user and a physical replication slot.

Then cloned the primary database to **srv-db2** using `pg_basebackup -R`, leaving it configured as a standby server that automatically follows the primary.

## Decisions

- Used **physical streaming replication** instead of logical replication because the goal was to replicate the entire PostgreSQL instance for high availability.
- Created a dedicated `replicator` user with the `REPLICATION` privilege instead of using the PostgreSQL superuser.
- Configured a physical replication slot (`srv_db2_slot`) so the primary keeps the required WAL files if the standby temporarily disconnects.

## Problems encountered

- **`pg_basebackup` connection timed out** because PostgreSQL was only listening on `localhost`. Fixed it by setting:

```text
listen_addresses = '*'
```

and restarting the PostgreSQL service.

- **Connection still timed out** after changing `listen_addresses`. The problem turned out to be UFW, which was blocking port **5432** by default. I fixed it by allowing PostgreSQL traffic only from the internal lab network:

```bash
sudo ufw allow from 192.168.56.0/24 to any port 5432 proto tcp
```

instead of opening the port to every address.

- **`pg_basebackup: directory ... exists but is not empty`** because the standby server still contained the default PostgreSQL cluster created during installation. I stopped PostgreSQL on **srv-db2**, removed the existing data directory and ran `pg_basebackup` again.

## Verification

Ran the following query on **srv-db**:

```sql
SELECT * FROM pg_stat_replication;
```

The output showed the `replicator` user connected from **192.168.56.13** with the state set to **streaming**, confirming that **srv-db2** was successfully replicating from the primary.