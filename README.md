# RHEL-Compatible Infrastructure Lab

> A hands-on learning project: building a small enterprise-style Linux datacenter from scratch using RHEL-compatible technologies. Built with AlmaLinux 9, BIND, chrony, Vagrant, and Ansible.

## Why this exists

I'm building this lab to develop real-world skills with the technologies used in enterprise Linux environments — specifically the RHEL family, DNS infrastructure, time synchronization, virtualization, and infrastructure automation. The aim is to understand each piece conceptually by building it by hand first, then layering on automation, so I learn both the concepts and the tools that implement them.

This is also a public portfolio for job applications in Linux systems engineering.

## Project goals

- [x] **Phase 1**: Authoritative DNS server with BIND 9 on AlmaLinux 9
- [x] **Phase 2**: NTP server with chrony, with offset/stratum monitoring
- [x] **Phase 3**: Multi-VM datacenter simulation with Vagrant + VirtualBox
- [ ] **Phase 4**: Full automation with Ansible (roles for DNS, NTP, hardening)
- [ ] **Phase 5**: CI with GitHub Actions, full documentation, migration plan

## Stack

| Component | Tool | Purpose |
|-----------|------|---------|
| Operating system | AlmaLinux 9 | RHEL-compatible Linux distribution |
| DNS server | BIND 9 | Authoritative DNS for an internal zone |
| Time sync | chrony | NTP server and client |
| Virtualization | VirtualBox + Vagrant | Multi-VM lab environment |
| Automation | Ansible | Idempotent configuration of all hosts |
| Version control | Git + GitHub | This repository |
| Documentation | Markdown | All docs in `docs/` |

## Status


**Phase 4** nearly complete. Ansible roles + site.yml orchestrator deploy the entire lab; disaster recovery proven (destroyed server, rebuilt in 60s via one command). Phase 5 (CI + docs) remaining. See [`docs/learning-journal.md`](docs/learning-journal.md) for day-by-day notes.

## Background

I'm transitioning into Linux systems engineering and using this lab to bridge the gap between tutorials and production-ready skills. The technologies chosen reflect what the target role (IT System Engineer Linux, RHEL/virtualization/DNS/NTP focus) actually uses on the job.

