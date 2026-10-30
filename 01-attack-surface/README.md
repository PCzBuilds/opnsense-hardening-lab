# 01 – Attack surface

**Goal:** Find out which services listen on the firewall and which of them can be reached from the LAN and from the WAN side.

## Steps

1. Open an SSH session to the firewall and choose option 8 (Shell) in the console menu.
2. List the listening IPv4 sockets.
3. From the LAN, test TCP ports 22, 53, 80 and 443 against the firewall.
4. From the upstream network, test the same ports against the WAN address.

## Commands

| Command | What it does |
|---|---|
| `ssh root@10.0.10.1` | `ssh` = SSH client, `root` = user, `@` separates user and target, `10.0.10.1` = LAN address of the firewall. |
| `sockstat -4 -l` | `sockstat` lists sockets of running programs. `-4` = IPv4 only. `-l` = only listening sockets. |
| `22,53,80,443 \| % { "$_ : " + (Test-NetConnection <target> -Port $_ -InformationLevel Quiet -WarningAction SilentlyContinue) }` | The list of ports is passed one by one (`\|`) to `ForEach-Object` (`%`). `$_` is the current port. `Test-NetConnection` tries a TCP connect; `-InformationLevel Quiet` returns only `True` or `False`; `-WarningAction SilentlyContinue` hides the warning on failure. |

How to read `sockstat`: `*` in the local address means all interfaces, `127.0.0.1` means local only. The foreign address of a listening socket is always `*:*`.

## Evidence

Listening services (PID and FD columns omitted, WAN address replaced):

| User | Command | Proto | Local address |
|---|---|---|---|
| unbound | unbound | udp4, tcp4 | `*:53` (listed twice per protocol) |
| unbound | unbound | tcp4 | `127.0.0.1:953` |
| nobody | dnsmasq | udp4 | `*:67` |
| nobody | dnsmasq | udp4, tcp4 | `*:53053` |
| root | ntpd | udp4 | `*:123`, `10.0.10.1:123`, `<WAN-IP>:123`, `127.0.0.1:123` |
| root | sshd | tcp4 | `10.0.10.1:22`, `127.0.0.1:22` |
| root | lighttpd | tcp4 | `*:443`, `*:80` |

Port test from the LAN against `10.0.10.1`:

```
22 : True
53 : True
80 : True
443 : True
```

Port test from the upstream network against `<WAN-IP>`:

```
22 : False
53 : False
80 : False
443 : False
```

The WAN address was read from the console menu (`WAN -> v4/DHCP4`). It also shows up in the `ntpd` line, because `ntpd` binds to every address of the firewall.

## Findings

- DNS resolver, web interface and several other services listen on all interfaces, including the WAN address. The WAN protection comes from the firewall rules (default deny), not from the services themselves.
- `sshd` is additionally bound to the LAN address and localhost. On the WAN address there is no SSH socket at all.
- SSH with the root password in the LAN is the weak spot. It is addressed in [02](../02-ssh-key-hardening/).

## Limits

- Four TCP ports only. UDP 53, 67, 123 and port 53053 were not tested.
- `False` means "no connection"; it does not say whether the packets were dropped or rejected.
- The WAN test ran from the upstream network segment only.
- Assumptions marked in the table: port 953 is probably the resolver's control channel, port 53053 probably the DNS function of `dnsmasq` on a fallback port because `unbound` uses port 53. Both were not verified.
