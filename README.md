# Small-Business Windows Domain: Build, Harden, Attack, Detect

A home-lab project that builds a realistic small-business Active Directory environment, hardens
it to a recognized baseline, instruments it with detection, and then attacks it to validate the
defenses. Everything runs in my own lab on isolated networking; there is no real or
third-party data involved.

**Why this project:** it demonstrates the full loop that most candidates only show half of.
I build and operate production infrastructure in my day-to-day work, and this lab lets me prove
that defensively (hardening, detection) *and* offensively (validating the hardening by attacking
it), which is the combination security teams value.

---

## What it demonstrates
- **Active Directory engineering:** domain design, OUs, GPOs, DNS/DHCP, least-privilege RBAC.
- **Infrastructure hardening** mapped to a recognized baseline (CIS Controls / NIST CSF).
- **Detection engineering:** a SIEM with agents, tuned rules, and dashboards for real attacks.
- **Offensive validation:** common AD attacks run against the lab, with before/after results.
- **Clear documentation:** the written report that makes all of the above legible to a reader.

## Architecture (lab)
```
Proxmox host
  ├─ pfSense / firewall ....... segmented VLANs, isolated from home network
  ├─ dc01 (Windows Server) .... Active Directory, DNS, DHCP
  ├─ win-client01 (Windows) ... domain-joined workstation (victim/user)
  ├─ srv01 (Linux) ............ file/app server (optional)
  ├─ siem01 (Wazuh) ........... log collection + detections + dashboard
  └─ kali01 (attacker) ........ offensive tooling, on a separate segment
```
All hostnames, IPs, and credentials are lab-only placeholders.

## Phased plan (built incrementally)

### Phase 1 - Build the domain
Stand up `dc01`, create the domain, add users/groups/OUs, join `win-client01`. Configure
DNS/DHCP. Baseline snapshot in Proxmox so I can reset between tests.

### Phase 2 - Harden to a baseline
Apply and document hardening mapped to **CIS Controls / NIST CSF**:
- Least-privilege RBAC; separate admin accounts; tiered admin model.
- GPO hardening: disable SMBv1, disable LLMNR/NBT-NS, enforce SMB signing, strong password /
  lockout policy, restrict NTLM where possible.
- **LAPS** for randomized local-admin passwords; **MFA** on remote access where applicable.
- Attack-surface reduction (AppLocker / ASR rules), patching cadence, host firewall.
- **Backups** with a 3-2-1 strategy (e.g., Proxmox Backup Server) and a tested restore.
- A short **security-program document** (WISP-style) mapping each control to its framework
  objective, written as a generic reference implementation.

### Phase 3 - Instrument detection
Deploy **Wazuh**: agents on `dc01` and `win-client01`, ship Windows Security logs, enable and
tune detections for the attacks in Phase 4 (failed-logon spikes, Kerberoasting, LLMNR
poisoning, new privileged account, suspicious service creation). Build a dashboard.

### Phase 4 - Attack and validate
From `kali01`, run representative AD attacks and record, for each: did it succeed, did the SIEM
detect it, and did the Phase 2 hardening prevent it?
- LLMNR/NBT-NS poisoning (Responder) and relay (ntlmrelayx).
- Kerberoasting; AS-REP roasting.
- Password spraying against the lockout policy.
- BloodHound enumeration of attack paths.
Run each **before and after** hardening to produce clear before/after evidence.

### Phase 5 - Write it up
The portfolio deliverable: architecture, the hardening decisions and *why*, the detections
built, the attacks run, and a **before/after table** showing which attacks were blocked or
detected after hardening. Screenshots throughout (safe to publish, since it is my own lab).

## Tooling
Proxmox, Windows Server, pfSense, Wazuh, Kali Linux, Responder, Impacket, BloodHound/SharpHound,
hashcat (offline cracking of captured hashes).

## Scope and ethics
Isolated lab, lab-only data, segmented from any real network. All identifiers are fictional.
The point is to demonstrate skills on a system I own, with nothing in the writeup that could be
used against a real environment.

---

*Status: in progress. This README is the project plan and will become the final writeup as each
phase is completed.*
