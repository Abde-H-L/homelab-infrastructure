# Phase 7 — Monitoring (srv-monitor)

## What was done

Installed **Node Exporter** on all 7 nodes using the `ansible/playbooks/node_exporter.yml` playbook. The playbook installs the service, enables it with systemd and opens port **9100** in UFW only for the internal network.

Updated the existing `ansible/playbooks/docker.yml` playbook to also install Docker on **srv-monitor**, avoiding a separate playbook.

Deployed **Prometheus** and **Grafana** on **srv-monitor** using `docker/monitoring/docker-compose.yml`. Prometheus was configured to scrape the Node Exporter endpoint from every machine in the lab, and Grafana was connected to Prometheus as its datasource. Finally, imported the **Node Exporter Full** community dashboard (ID **1860**).

## Decisions

- Reused the existing `docker.yml` playbook instead of creating another one just for **srv-monitor**.
- Configured both Prometheus and Grafana with `network_mode: host` instead of Docker's default bridge network after running into networking issues between the containers and the host. For this lab, it was a simpler and more reliable solution.
- Imported the community **Node Exporter Full** dashboard instead of creating one from scratch, since it already provides useful CPU, RAM, disk and network metrics.

## Problems encountered

- **srv-monitor's own Node Exporter target stayed DOWN**, while every other node was reachable. The problem was caused by Docker's bridge network, which couldn't reliably reach a service running on the same host.
- Tried using the Docker bridge gateway address (`172.17.0.1`) instead of the server's LAN IP. It worked from the host itself, but Prometheus running inside the container still couldn't reach it consistently.
- Solved the issue by switching both **Prometheus** and **Grafana** to `network_mode: host`, allowing them to communicate directly through the host's network stack.
- **UFW** was also blocking port **9100** by default. Added a rule to allow Node Exporter traffic only from the internal network (`192.168.56.0/24`).

## Verification

Checked **Prometheus → Status → Targets** and confirmed that all seven **Node Exporter** targets were **UP**.

Verified that Grafana connected successfully to Prometheus, and the imported **Node Exporter Full** dashboard displayed live CPU, memory, disk and network metrics for every machine in the homelab.