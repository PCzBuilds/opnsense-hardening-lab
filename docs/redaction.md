# Redaction

The exercises were done on a real device in a lab. Values that could identify the environment were removed or replaced before publishing. Structure and results are unchanged.

| What | Treatment |
|---|---|
| Addresses on the upstream (WAN) side | Replaced by `<WAN-IP>`. |
| LAN addresses | Replaced by `10.0.10.0/24` (firewall `10.0.10.1`, workstation `10.0.10.50`). The addresses in the lab were different. |
| Source ports and log timestamps | Source ports changed; timestamps replaced by offsets in seconds. |
| Network interface names | Not included. |
| SSH host key fingerprints and the HTTPS certificate fingerprint | Not included. |
| Fingerprint of the generated key | Not included. |
| Passwords | Never shown. |
| Windows user name in file paths | Replaced by `<user>`. |
| Device vendor and model | Not named. The device is described as an x86 firewall appliance. |

Public values are kept as they are: the test target `1.1.1.1`, the name `example.org` and the DNS answers for it. The admin user name `labadmin` and the key label `opnsense-lab` are lab labels.

## Why there are no screenshots

Photos of a screen leak information in two ways: visible text, and text that stays readable when a redaction box is only drawn on top of an image. A black box only protects if it is burned into the pixels. To avoid the risk, this repository contains no images. Outputs are reproduced as text and trimmed, and every trimming is stated.
