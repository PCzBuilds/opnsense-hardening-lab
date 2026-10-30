# 02 – SSH key-only login

**Goal:** SSH login only with a key and a personal admin user. Root login and password login are switched off.

**Order matters:** create the key, create the user with the public key, test the key login, and only then switch off root and password login. The web interface stays open in a browser tab the whole time as a second way in.

## Steps

1. Create an ed25519 key pair on the workstation (with passphrase).
2. In the web interface: System > Access > Users, add a user (table below).
3. Test the key login.
4. System > Settings > Administration > Secure Shell: switch off "Permit root user login" and "Permit password login", save.
5. Run three tests.

## Commands

| Command | What it does |
|---|---|
| `mkdir -Force "$env:USERPROFILE\.ssh"` | `mkdir` creates a folder, `-Force` = no error if it exists, `$env:USERPROFILE` = user folder, `\.ssh` = folder where SSH expects keys. |
| `ssh-keygen -t ed25519 -C "opnsense-lab" -f "$env:USERPROFILE\.ssh\id_ed25519_opn"` | `-t ed25519` = key type, `-C` = comment label on the key, `-f` = path and file name. Creates `id_ed25519_opn` (private) and `id_ed25519_opn.pub` (public). The passphrase protects the private key. |
| `Get-Content "$env:USERPROFILE\.ssh\id_ed25519_opn.pub" \| clip` | Reads the public key and copies it to the clipboard (`clip`). |
| `ssh -i "$env:USERPROFILE\.ssh\id_ed25519_opn" labadmin@10.0.10.1` | `-i` = identity file (the private key). |
| `ssh -o PubkeyAuthentication=no labadmin@10.0.10.1` | `-o` sets one SSH option. `PubkeyAuthentication=no` switches the key off for this attempt, so only the password is left. |

## User settings

| Field | Value | Why |
|---|---|---|
| Username | `labadmin` | Personal admin account instead of sharing root. |
| Password | (strong, not shown) | Needed for the web interface only. |
| Login shell | `/bin/sh` | With `nologin` the user cannot log in via SSH. |
| Group membership | `admins` | Administrator rights in the management interface. |
| Authorized keys | `ssh-ed25519 AAAA… opnsense-lab` | The public key, one line. The private key stays on the workstation. |

SSH settings after the change: Enable Secure Shell on, Permit root user login off, Permit password login off, SSH port 22, Listen interfaces LAN.

## Evidence

Key creation (fingerprint omitted):

```
PS> ssh-keygen -t ed25519 -C "opnsense-lab" -f "$env:USERPROFILE\.ssh\id_ed25519_opn"
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\<user>\.ssh\id_ed25519_opn
Your public key has been saved in C:\Users\<user>\.ssh\id_ed25519_opn.pub
```

Key login before the change:

```
PS> ssh -i "$env:USERPROFILE\.ssh\id_ed25519_opn" labadmin@10.0.10.1
Enter passphrase for key 'C:\Users\<user>\.ssh\id_ed25519_opn':
labadmin@OPNsense:~ $ whoami
labadmin
```

Tests after the change:

| Test | Expected | Result |
|---|---|---|
| `ssh -i <key> labadmin@10.0.10.1` | login works | login works (passphrase prompt, shell prompt) |
| `ssh root@10.0.10.1` | refused | `root@10.0.10.1: Permission denied (publickey).` |
| `ssh -o PubkeyAuthentication=no labadmin@10.0.10.1` | refused | `labadmin@10.0.10.1: Permission denied (publickey).` |

## Findings

- The server now offers only `publickey`. A password is no longer an option for any user.
- The message for `root` alone does not tell whether root login is off or the key was wrong. Together with the third test it shows that password login is gone.
- Fix that was needed on the way: a typo `ssh -keygen` instead of `ssh-keygen` produced `Bad escape character 'ygen'`, because `ssh` read `-keygen` as options.

## Limits

- Not tested: whether the brute-force lockout (`sshlockout`) counts failed key logins. Without passwords there are no wrong passwords left to test with, and a test could lock out the workstation.
- The admin user has no `sudo` configuration here; the exercise covers login only.
