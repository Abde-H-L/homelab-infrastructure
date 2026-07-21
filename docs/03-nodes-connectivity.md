# Phase 3 — Remaining Nodes and Connectivity

## What was done
Cloned the 6 remaining nodes from the base template (`srv-ops`, `srv-web`, `srv-db`, `srv-db2`, `srv-monitor` and `srv-backup`). Configured a static IP address and hostname for each node, using `srv-gw` as the default gateway and DNS server.

## Decisions
I followed a simple IP addressing scheme, assigning a fixed IP address to each server based on its role.

| Node | IP Address |
|------|------------|
| `srv-ops` | `192.168.56.10/24` |
| `srv-web` | `192.168.56.11/24` |
| `srv-db` | `192.168.56.12/24` |
| `srv-db2` | `192.168.56.13/24` |
| `srv-monitor` | `192.168.56.14/24` |
| `srv-backup` | `192.168.56.15/24` |

I also used simple and descriptive hostnames based on each server's role.

## Problems encountered
The only issues were a few typos while configuring the IP addresses and the Netplan files. I identified the wrong IP using `ip a`, corrected it, and fixed the Netplan spelling errors after `sudo netplan apply` reported them.

## Verification
All nodes can successfully ping `srv-gw` and reach the Internet (`8.8.8.8`) through it. They can also communicate with each other on the internal network.