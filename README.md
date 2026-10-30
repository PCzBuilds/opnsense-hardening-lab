# opnsense-hardening-lab

Three small, verifiable hardening exercises on a default OPNsense firewall in a lab. Every exercise follows the same pattern: measure, change, test, roll back.

> Lab only: a private test LAN with one workstation, no production systems. Addresses on the upstream side, key material and host fingerprints are removed or replaced. See [docs/redaction.md](docs/redaction.md).

## Question

Does a default OPNsense install expose more than necessary, and can a few small changes reduce that in a way that can be proven with measurements?

## Lab setup

- OPNsense 26.7 (amd64) on an x86 firewall appliance
- One Windows workstation on the LAN (10.0.10.0/24, addresses from the firewall's DHCP server)
- WAN as DHCP client in an upstream network (addresses omitted)
- Tools: `sockstat` (FreeBSD), PowerShell (`Test-NetConnection`, `Resolve-DnsName`), OpenSSH client, OPNsense Live View

```mermaid
flowchart LR
  PC["Workstation (LAN client)"] --- FW["OPNsense 26.7<br/>LAN 10.0.10.1"] --- UP["Upstream network / Internet<br/>(WAN, DHCP)"]
```

## Results

| # | Exercise | Change | Proof |
|---|----------|--------|-------|
| 1 | [Attack surface](01-attack-surface/) | none (measurement) | `sockstat -4 -l` lists the listening services. Ports 22, 53, 80, 443: reachable from LAN, not reachable from WAN. |
| 2 | [SSH key-only login](02-ssh-key-hardening/) | own admin user with ed25519 key; root login and password login switched off | `root` and the admin user without key both get `Permission denied (publickey)`; key login still works. |
| 3 | [Egress filter](03-egress-filter/) | port alias (53, 80, 443) with a pass rule and a block-rest rule on LAN | Ping and port 853 blocked, port 443 and DNS still work; blocks visible in the live log; rules removed afterwards. |

## Limits

- Lab scale: only four TCP ports were tested; UDP services (53, 67, 123) and port 53053 were not.
- A failed test (`False`) means "no connection", not "dropped" or "rejected".
- The egress filter covers IPv4 only. The IPv6 default-allow rule stayed active.
- Not tested: whether the brute-force lockout (`sshlockout`) counts failed key logins. A test could lock out the admin workstation, so it was left out on purpose.

## Repository layout

```
opnsense-hardening-lab/
├── README.md
├── 01-attack-surface/README.md
├── 02-ssh-key-hardening/README.md
├── 03-egress-filter/README.md
└── docs/
    ├── method.md
    └── redaction.md
```

## Method

Short version: baseline measurement, one change, test with the same measurement, rollback. Details and evidence rules: [docs/method.md](docs/method.md).

Exercises done in 2026. Author: PCzBuilds
