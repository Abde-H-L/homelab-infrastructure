# Phase 2 — Internal Network and srv-gw

## What was done
Configured `srv-gw` as the gateway between the VirtualBox NAT adapter and the internal network. I configured the static IP with Netplan, enabled IP forwarding and added the NAT rules using nftables.

## Decisions
I decided to use nftables instead of iptables because it is the current standard in Linux and it is easier to manage. I also chose the `192.168.56.0/24` network and assigned `192.168.56.1` to `srv-gw`.

## Problems encountered
I had a typo in the `nftables.conf` file (`protocool` instead of `protocol`), so the service did not start. I checked the error with `systemctl status nftables`, fixed it and verified the configuration with `nft -c -f`.

## Verification
Both network interfaces are configured correctly (`ip a`), the nftables rules are loaded (`nft list ruleset`) and the machines on the internal network can access the Internet through `srv-gw`.