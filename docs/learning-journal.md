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
