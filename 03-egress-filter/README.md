# 03 – Egress filter with a port alias

**Goal:** The LAN may send only DNS (53), HTTP (80) and HTTPS (443) to the outside. Everything else is blocked and logged. The rules are removed afterwards.

An alias is a named list that can be reused in rules. Here it holds the allowed ports.

## Steps

1. Baseline measurement.
2. Firewall > Aliases: create the alias, save, apply.
3. Firewall > Rules > LAN: disable the IPv4 default-allow rule (do not delete it), add the pass rule and the block rule, apply.
4. Measure again, read the live log.
5. Roll back: delete both rules, enable the default rule again, apply, check with a ping.

## Alias

| Field | Value |
|---|---|
| Name | `ALLOWED_PORTS` |
| Type | Port(s) |
| Content | `53`, `80`, `443` (three separate entries) |
| Description | Egress allowed: DNS, HTTP, HTTPS |

Entering `53, 80, 443` as one string failed with `Entry " 80" is not a valid port number.` because the space after the comma stayed in the entry. Each port has to be entered on its own, without spaces.

## Rules (LAN, direction in)

| Rule | Settings |
|---|---|
| Default allow LAN to any (IPv4) | disabled, not deleted |
| 1: "Egress: allowed ports" | Pass, IPv4, TCP/UDP, source LAN net, destination any, destination port `ALLOWED_PORTS` |
| 2: "Egress: block rest" | Block, IPv4, any protocol, source any, destination any, logging on |

The order matters: the pass rule has to come before the block rule. The IPv6 default-allow rule stayed active.

## Commands

| Command | What it does |
|---|---|
| `ping 1.1.1.1` | Sends echo requests (4 on Windows) to a public DNS server, used as an internet target. |
| `443,853 \| % { "$_ : " + (Test-NetConnection 1.1.1.1 -Port $_ -InformationLevel Quiet -WarningAction SilentlyContinue) }` | Same port loop as in exercise 1, target `1.1.1.1`, ports 443 (HTTPS) and 853 (DNS over TLS). |
| `Resolve-DnsName example.org` | Asks DNS for the addresses of a name (A = IPv4, AAAA = IPv6). |

## Evidence

| Measurement | Ping 1.1.1.1 | Port 443 | Port 853 | DNS query |
|---|---|---|---|---|
| Before (no rules) | 4 of 4 | True | True | not tested |
| With egress filter | 0 of 4 (100 % loss) | True | False | works |
| After rollback | 4 of 4 | not tested | not tested | not tested |

DNS with the filter active returned A records (`104.20.26.136`, `172.66.157.237`) and AAAA records for `example.org`.

Live log, quick search `1.1.1.1` (times shown as offsets from the first entry of each test):

| Offset | Interface | Proto | Source | Destination | Action | Label |
|---|---|---|---|---|---|---|
| +0 s | LAN | TCP | 10.0.10.50:50123 | 1.1.1.1:853 | block | Egress: block rest |
| +1 s | LAN | TCP | 10.0.10.50:50123 | 1.1.1.1:853 | block | Egress: block rest |
| +3 s | LAN | TCP | 10.0.10.50:50123 | 1.1.1.1:853 | block | Egress: block rest |
| +7 s | LAN | TCP | 10.0.10.50:50123 | 1.1.1.1:853 | block | Egress: block rest |
| +14 s | LAN | TCP | 10.0.10.50:50123 | 1.1.1.1:853 | block | Egress: block rest |
| +0 s to +14 s (separate ping test) | LAN | ICMP | 10.0.10.50 | 1.1.1.1 | block | Egress: block rest (4 entries, about 5 s apart) |

The five TCP entries are connection retries with growing gaps. The four ICMP entries match the four ping timeouts. The HTTPS test is not in the log because the pass rule does not log.

## Findings

- The filter does what it should: only 53, 80 and 443 leave the LAN over IPv4.
- A pass rule for the wanted ports above a logged block rule keeps the rule set short and makes denied traffic visible.
- The live log first showed only WAN noise (broadcasts, the firewall's own DNS lookups). A quick search on the target address made the blocks visible.

## Limits

- IPv4 only; the IPv6 default-allow rule was untouched.
- The block rule's source was `any` instead of `LAN net`. This makes no difference on the LAN ingress, because only LAN sources arrive there.
- Allowing 443 to anywhere still lets any HTTPS service through. The filter limits ports, not destinations.
