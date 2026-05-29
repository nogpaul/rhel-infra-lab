# Linux Learning Journal

## Day 1 — May 26, 2026

### What I learned
- AlmaLinux 9 is a free, RHEL-compatible Linux distribution 
	"AlmaLinux 9 is a free, binary-identical rebuild of RHEL 9. 
	The job asks for RHEL experience; RHEL costs money; AlmaLinux gives me the same skills for free
- The RHEL family uses .rpm packages, managed by `dnf`
- File permissions are 3 groups (owner, group, others) × 3 bits (r, w, x)
- `sudo` runs a single command as root (the superuser)
- I updated my system from 9.7 to 9.8 using `sudo dnf update`

### Things I want to understand better
- (Why AlmaLinux 9 When and how to change users)

## Day 2 — May 27, 2026

### What I built
- Set up the project on GitHub with proper structure (README, .gitignore, docs/, configs/)
- Generated SSH keys and connected AlmaLinux to GitHub securely
- Installed and configured BIND 9 as an authoritative DNS server for `lab.local`
- Created zone files with SOA, NS, and A records for `dns`, `web`, `ntp`
- Service is enabled and running; queries via `dig` return correct authoritative answers

### What I learned
- **SSH keys** — asymmetric crypto: public key shared, private key never leaves my machine. Proof of identity without sending the secret.
- **The service abstraction** — 9 properties an OS provides for any service. systemd is the unified solution to all 9.
- **systemctl** — `enable` (boot) vs `start` (now); `enable --now` does both; `static` means runs only when something else needs it.
- **DNS conceptually** — distributed, hierarchical, cached, redundant key-value lookup with delegation. The shape of DNS is a direct response to the problems HOSTS.TXT had.
- **Authoritative vs recursive** — authoritative owns data; recursive resolves. My lab is authoritative for `lab.local`.
- **Zone file format** — SOA structure (serial, refresh, retry, expire, minimum), `@` shorthand, and the **trailing-dot rule** (FQDN ends with `.`; relative names get zone appended).
- **Validation first** — `named-checkconf` and `named-checkzone` catch errors before systemd will even try to start.
- **`dig` reading** — the `aa` flag means authoritative answer; the "recursion requested but not available" warning *confirms* recursion-disabled is working.

### Phase 1 status
**Complete.** Authoritative DNS for `lab.local` running, three test records pointing to 127.0.0.1 (will become real VM IPs in Phase 3).

### Open questions / revisit later
- DNSSEC validation chain
- mDNS conflict implications of `.local`
- Why BIND uses 241MB memory for a tiny zone

## Day 3 — May 28, 2026

### What I built
- Installed chrony (NTP daemon) on AlmaLinux 9
- Verified it as a client: synced to a stratum-0 PTP hardware clock via WSL2 (microsecond accuracy)
- Configured it as a time *server* (`allow` + `local stratum 10`)
- Confirmed it listens on UDP 123 to serve clients

### What I learned
- **Concept vs tool again**: NTP is the system; chrony is one implementation (like DNS↔BIND). Swappable for ntpd or systemd-timesyncd.
- **Why clocks must agree**: not about one clock drifting — machines *disagreeing* breaks auth (Kerberos/TLS validity windows), log correlation, distributed data.
- **Stratum hierarchy**: stratum 0 (atomic/GPS) → 1 → 2 → … Same hierarchical-delegation shape as DNS.
- **NTP uses DNS**: chrony resolves pool.ntp.org names to find its time servers. Layers depend on layers.
- **Stepping vs slewing**: jump vs gradual adjust. `makestep 1.0 3` = jumps only in first 3 updates.
- **driftfile**: chrony learns and persists the clock's drift rate; read it back on restart.
- **Two clocks**: system clock (RAM) vs hardware/RTC clock. `rtcsync` aligns them.
- **rpm vs dnf**: dnf = high-level (repos, dependencies); rpm = low-level local query. Same as apt/dpkg.
- **chronyc** = dig-for-time. **ss** for sockets: 123 = NTP service, 323 = local control. **Reach 377** (octal) = perfect reachability.

### Pattern reinforcement (recurred from Phase 1)
- Dedicated `chrony` user (least privilege); config in /etc/ (config-as-code)
- Package name (chrony) ≠ service name (chronyd); enabled vs started; all 9 service properties present

### Phase 2 status
**Complete.** chrony running as NTP client (synced) and server (listening on 123, ready for VMs in Phase 3).

## Day 4 — May 29, 2026
### What I built
- Spun up a 3-VM datacenter on Vagrant + VirtualBox: server (DNS+NTP) and two clients on private network 192.168.56.0/24
- Deployed the BIND and chrony configs from Phase 1/2 onto the server VM (192.168.56.10), updated for the new IP/subnet
- Configured both clients to use the lab DNS server and sync time from the lab NTP server
- Added BIND forwarders (8.8.8.8) so the lab DNS resolves external names too — clients now use **only** the lab DNS, no fallback needed
### What I learned
- **Hypervisor concept**: VirtualBox is a type-2 hypervisor. Vagrant is the declarative front-end — the Vagrantfile *is* the datacenter, in code.
- **Infrastructure-as-code for VMs**: `vagrant up` reads the Vagrantfile and reproducibly builds the environment. Same idea as Terraform but for local VMs.
- **Network modes**: NAT (eth0, internet access, isolated per-VM) vs private/host-only network (eth1, VMs see each other). Two adapters, two purposes.
- **Vagrant box** = pre-built VM image, cached after first download. The almalinux/9 box is the template.
- **Servers ship lean**: had to install `bind-utils` (for dig) and `nano` on the clients — they don't come with anything you don't need. Real production posture.
- **Package vs service (live again)**: `chrony` is the package, `chronyd` is the service. `bind` package → `named` service.
- **DNS isolation problem**: an authoritative-only DNS knows only its zones. Without forwarders, clients couldn't resolve `google.com` through it. Two fixes: (a) fallback DNS on each client, or (b) add forwarders on the lab DNS so it forwards what it doesn't know. (b) is the right design — single authoritative point.
- **DNSSEC trust chain**: enabling `recursion yes` + `forwarders` initially gave SERVFAIL because BIND tried to validate the DNSSEC chain and broke. Disabled `dnssec-validation` for the lab. In production you'd configure trust anchors properly.
- **`forward only;`** keeps BIND from falling back to root-server recursion when the forwarder doesn't answer — locks behavior to the forwarder.
- **NTP stratum hierarchy (live)**: server VM ran at stratum 4 (synced from a stratum-3 internet source); clients showed stratum 5 — exactly server+1. The hierarchy printed itself on my terminal.
- **chronyc sources flags**: `^?` = source unknown/not selected, `^*` = currently selected synced source. Multiple sources with `^?` = confused/unsynced; one `^*` = healthy.
- **resolv.conf is fragile**: NetworkManager rewrites it on network events. Edits survive only until next refresh. Persistent fix = `nmcli` (or Ansible in Phase 4).
### Pattern reinforcement (recurred from Phase 1/2)
- Same config-as-code workflow, just deployed to remote VMs via shared folder
- Same package/service split, same systemd enable+start
- Same "permission denied on /etc/named.conf" reflex → `sudo`
### Troubleshooting log (worth remembering)
- chrony client showed multiple `^?` sources and `Stratum 0 / Not synchronised` → cause: forgot to comment out the default `pool` line when adding `server 192.168.56.10`. Two competing sources confused it. Single source → `^*` immediately.
- BIND SERVFAIL on `dig google.com` → cause: DNSSEC chain validation broken via forwarder. Fix: `dnssec-validation no;` in lab.
### Phase 3 status
**Complete.** Three-VM lab running. DNS + NTP serving both clients. Lab DNS forwards external queries. Ready for Phase 4 (Ansible automation).
