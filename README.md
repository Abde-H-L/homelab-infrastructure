# homelab-infrastructure

## Description
Enterprise infrastructure laboratory developed during the 1st year of ASIR (Administración de Sistemas Informáticos en Red).

## Objectives
- Build an isolated network environment.
- Implement a secure gateway mechanism.
- Automate configuration management using Ansible.
- Deploy services encapsulated within Docker containers.
- Implement high availability across database and application nodes.
- Monitor infrastructure health and performance metrics.
- Apply a robust and consistent backup strategy.
- Reproduce parts of the environment inside Azure.

## Architecture

```text
   Internet
      │
VirtualBox NAT
      │
   srv-gw
      │
── ── ┴ ── ── ── ── ── ── ── ── ── ── ── ── ── ──
   │        │        │        │        │        │
  ops      web       db      db2      mon     backup
```

## Technologies Used

Here is the stack I used to build and deploy this infrastructure:

| Technology | Use |
| :--- | :--- |
| **Ubuntu Server** | Used as the base operating system for all my infrastructure nodes. |
| **VirtualBox** | Used as the local hypervisor to virtualize the private and isolated network. |
| **Ansible** | Used to automate configuration management and provisioning across all nodes. |
| **Docker** | Used to containerize and deploy all the application services. |
| **PostgreSQL** | Deployed as the main relational database engine with high availability. |
| **Prometheus** | Implemented to collect metrics and handle time-series monitoring backend. |
| **Grafana** | Used to build dashboards and visualize system metrics in real time. |
| **Terraform** | Used as the Infrastructure as Code (IaC) tool to provision the environment. |
| **Azure** | Used as the cloud provider to replicate and test part of the environment. |