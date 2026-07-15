# SSH Key Authentication for Disabled Users — The Complete Guide

> **Author:** Jack Liu Shurui — **Repository:** https://github.com/jackliusr/research — **Date:** July 2026

---

## 1. The Problem: What "Disabled User" Really Means

SSH key authentication with a "disabled" user is common in production, but "disabled" is ambiguous:

| State | Mechanism | Effect on SSH Key Auth |
|---|---|---|
| **Disabled shell** | Shell set to `/sbin/nologin` or `/bin/false` | **Works** for port forwarding; interactive login and commands fail |
| **Locked password** | `passwd -l` or `usermod -L` | **Works** — lock affects PAM password auth only, not publickey |
| **Expired password** | `chage -d 0` or `passwd -e` | **Works** — key auth bypasses password expiry |
| **sshd_config DenyUsers** | Explicit block in config | **Does NOT work** — DenyUsers blocks all auth methods |
| **Account expired** | `usermod --expiredate 1` | **Works** (with valid shell) — expiration is PAM-level |

**Core insight:** OpenSSH public key authentication is handled internally by sshd, **not through PAM or the user's shell**. The shell and password lock are checked *after* authentication, during session setup. Key auth works for users who cannot log in interactively, cannot use passwords, or have no valid shell.

---

## 2. User Account with Disabled Shell (nologin)

### 2.1 How It Works

When a user's shell is `/sbin/nologin` (or `/usr/sbin/nologin` or `/bin/false`):
- sshd performs public key auth **without checking the shell** — it validates the key against `authorized_keys`, then tries to execute the shell.
- **Interactive session** (`ssh user@host`): nologin prints a message and exits.
- **Command execution** (`ssh user@host "cmd"`): nologin exits before the command runs.
- **Port forwarding** (`ssh -N -L ...`): **works** — no shell needed.

| Shell | Interactive | `ssh user@host "cmd"` | scp/sftp | Port forwarding |
|---|---|---|---|---|
| `/sbin/nologin` | Fails | Fails | Fails | **Works** |
| `/bin/false` | Fails | Fails | Fails | **Works** |

### 2.2 Setup

```bash
sudo useradd -m -s /sbin/nologin svc_backup
sudo usermod -s /sbin/nologin existinguser
grep "^existinguser:" /etc/passwd
ssh existinguser@localhost                    # Fails
ssh -N -L 8080:localhost:80 existinguser@localhost   # Works
```

### 2.3 Making Command Execution Work

Use a wrapper script as the shell:

```bash
sudo tee /usr/local/bin/rssh.sh << 'EOF'
#!/bin/bash
if [ -n "$SSH_ORIGINAL_COMMAND" ]; then exec /bin/bash -c "$SSH_ORIGINAL_COMMAND"
else echo "Interactive login disabled."; exit 1; fi
EOF
sudo chmod 755 /usr/local/bin/rssh.sh
sudo usermod -s /usr/local/bin/rssh.sh svc_deploy
echo "/usr/local/bin/rssh.sh" | sudo tee -a /etc/shells
ssh svc_deploy@host "uptime"  # Works
ssh svc_deploy@host            # Fails
```

---

## 3. User Account with Locked Password

`passwd -l` prepends `!!` to the password hash in `/etc/shadow`. **SSH key auth bypasses PAM entirely** — sshd validates the key signature directly against `authorized_keys`, so the password lock has no effect on key auth.

```bash
sudo passwd -l svc_deploy
sudo passwd -S svc_deploy       # Shows "L" for locked
sudo grep "^svc_deploy:" /etc/shadow  # Shows "!!" prefix
ssh -i ~/.ssh/svc_key svc_deploy@host                 # Success
ssh -o PreferredAuthentications=password svc_deploy@host  # Denied

# Best practice: combine locked password + nologin + SSH key
sudo useradd -m -s /sbin/nologin svc_backup
sudo passwd -l svc_backup
sudo mkdir -p ~svc_backup/.ssh
echo "ssh-ed25519 AAAA..." | sudo tee ~svc_backup/.ssh/authorized_keys
sudo chmod 700 ~svc_backup/.ssh && sudo chmod 600 ~svc_backup/.ssh/authorized_keys
sudo chown -R svc_backup:svc_backup ~svc_backup/.ssh
```

---

## 4. sshd_config Restrictions and Directives

### 4.1 Directives That Affect Key Auth

| Directive | Effect on Key Auth | Effect on Password Auth |
|---|---|---|
| `PasswordAuthentication no` | **None** | Disabled |
| `PubkeyAuthentication yes` | **Required** | N/A |
| `DenyUsers user` | **Blocks all auth** | Blocks all auth |
| `AllowUsers user` | **Whitelist** | Same |
| `AuthenticationMethods publickey` | **Only key allowed** | Disabled |

**CRITICAL:** `DenyUsers` and `DenyGroups` block **all** authentication methods, including publickey.

### 4.2 AuthenticationMethods and Match Blocks

```bash
# Global — require publickey
AuthenticationMethods publickey

# Per-user overrides
Match User svc_admin
    AuthenticationMethods publickey,password   # Both required
Match User svc_deploy
    AuthenticationMethods publickey             # Key only

# Match by group or address
Match Group automation
    PasswordAuthentication no
    AuthenticationMethods publickey
Match Address 10.0.0.0/8
    PasswordAuthentication no
    AuthenticationMethods publickey
```

### 4.3 Allow/Deny Processing Order

1. `DenyUsers` — matched = **denied** regardless of auth method
2. `DenyGroups` — matched = **denied**
3. `AllowUsers` — if set, only matching users permitted
4. `AllowGroups` — if set, only matching groups permitted

### 4.4 Disabling Password Auth Globally

```bash
# /etc/ssh/sshd_config
PasswordAuthentication no
ChallengeResponseAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PermitRootLogin prohibit-password
sudo systemctl restart sshd
```

---

## 5. Complete Scenarios

### Scenario A — Service Account for Automation

```bash
sudo useradd -m svc_deploy
sudo tee /usr/local/bin/cmd-shell.sh << 'SCRIPT'
#!/bin/bash
if [ -n "$SSH_ORIGINAL_COMMAND" ]; then exec /bin/bash -c "$SSH_ORIGINAL_COMMAND"
else echo "Interactive login disabled."; exit 1; fi
SCRIPT
sudo chmod 755 /usr/local/bin/cmd-shell.sh
sudo usermod -s /usr/local/bin/cmd-shell.sh svc_deploy
echo "/usr/local/bin/cmd-shell.sh" | sudo tee -a /etc/shells
sudo passwd -l svc_deploy
sudo mkdir -p ~svc_deploy/.ssh && sudo chmod 700 ~svc_deploy/.ssh
echo "ssh-ed25519 AAAA..." | sudo tee ~svc_deploy/.ssh/authorized_keys
sudo chmod 600 ~svc_deploy/.ssh/authorized_keys
sudo chown -R svc_deploy:svc_deploy ~svc_deploy/.ssh
# ssh svc_deploy@host "uptime" ✓ | ssh svc_deploy@host ✗
```

### Scenario B — Git/rsync-Only User

```bash
# Git-only user
sudo useradd -m -s /usr/bin/git-shell gituser && sudo passwd -l gituser
sudo mkdir -p ~gituser/.ssh && sudo chmod 700 ~gituser/.ssh
echo "ssh-ed25519 AAAA..." | sudo tee ~gituser/.ssh/authorized_keys
sudo chmod 600 ~gituser/.ssh/authorized_keys && sudo chown -R gituser:gituser ~gituser/.ssh
echo "/usr/bin/git-shell" | sudo tee -a /etc/shells
# git clone gituser@server:repo.git ✓ | ssh gituser@server ✗

# rsync-only user (command= restriction in authorized_keys)
sudo useradd -m -s /sbin/nologin rsyncuser && sudo passwd -l rsyncuser
sudo mkdir -p ~rsyncuser/.ssh && sudo chmod 700 ~rsyncuser/.ssh
echo 'command="/usr/bin/rsync --server -vlogDtprze.iLsfxC . /data/archive",no-port-forwarding,no-agent-forwarding ssh-ed25519 AAAA...' | sudo tee ~rsyncuser/.ssh/authorized_keys
sudo chmod 600 ~rsyncuser/.ssh/authorized_keys && sudo chown -R rsyncuser:rsyncuser ~rsyncuser/.ssh
# rsync -avz file rsyncuser@server:/data/archive/ ✓ | ssh rsyncuser@server ✗
```

### Scenario C — Chrooted SFTP-Only User

```bash
sudo groupadd sftpusers && sudo useradd -m -s /sbin/nologin -g sftpusers sftp_user
sudo passwd -l sftp_user
sudo mkdir -p /srv/sftp/chroot/uploads
sudo chown root:root /srv/sftp/chroot && sudo chmod 755 /srv/sftp/chroot
sudo chown sftp_user:sftpusers /srv/sftp/chroot/uploads
sudo mkdir -p ~sftp_user/.ssh && sudo chmod 700 ~sftp_user/.ssh
echo "ssh-ed25519 AAAA..." | sudo tee ~sftp_user/.ssh/authorized_keys
sudo chmod 600 ~sftp_user/.ssh/authorized_keys && sudo chown -R sftp_user:sftpusers ~sftp_user/.ssh

# In /etc/ssh/sshd_config:
#   Subsystem sftp internal-sftp
#   Match Group sftpusers
#       ChrootDirectory /srv/sftp/chroot
#       ForceCommand internal-sftp
#       X11Forwarding no / AllowTcpForwarding no
#       PasswordAuthentication no / AuthenticationMethods publickey
sudo systemctl restart sshd
# sftp -i key sftp_user@server ✓ | ssh sftp_user@server ✗
```

### Scenario D — Tunnel-Only Jump Host User

```bash
sudo useradd -m -s /sbin/nologin tunneluser && sudo passwd -l tunneluser
sudo mkdir -p ~tunneluser/.ssh && sudo chmod 700 ~tunneluser/.ssh
echo 'no-pty,no-agent-forwarding,no-X11-forwarding,permitopen="localhost:5432" ssh-ed25519 AAAA...' | sudo tee ~tunneluser/.ssh/authorized_keys
sudo chmod 600 ~tunneluser/.ssh/authorized_keys && sudo chown -R tunneluser:tunneluser ~tunneluser/.ssh
# ssh -N -L 5432:localhost:5432 tunneluser@jumphost ✓
# ssh tunneluser@jumphost ✗
```

---

## 6. Key Configuration Points

### 6.1 File Permissions (Most Common Pitfall)

```bash
~/.ssh/                   # 0700 (drwx------), owned by user
~/.ssh/authorized_keys    # 0600 (-rw-------), owned by user
Home directory            # NOT group-writable or world-writable
```

sshd `StrictModes` (enabled by default) rejects auth if permissions are wrong: `Authentication refused: bad ownership or modes`. Fix:

```bash
chmod go-w ~user && chmod 700 ~user/.ssh && chmod 600 ~user/.ssh/authorized_keys
chown -R user:user ~user/.ssh
sudo restorecon -R ~user/.ssh   # SELinux only
```

### 6.2 AuthorizedKeysFile Location

```bash
# Default (per-user)
AuthorizedKeysFile .ssh/authorized_keys
# Central location
AuthorizedKeysFile /etc/ssh/authorized_keys/%u

sudo mkdir -p /etc/ssh/authorized_keys
echo "ssh-ed25519 AAAA..." | sudo tee /etc/ssh/authorized_keys/svc_deploy
sudo chmod 644 /etc/ssh/authorized_keys/svc_deploy
```

### 6.3 UsePAM and /etc/shells

With `UsePAM yes` (default), key auth works independently — sshd validates keys internally. PAM is only consulted for password auth, challenge-response, and session setup.

sshd also checks that the user's shell appears in `/etc/shells`:
```
User svc_deploy not allowed because shell /usr/local/bin/cmd-shell.sh does not exist
```
Fix: `echo "/usr/local/bin/cmd-shell.sh" | sudo tee -a /etc/shells`

---

## 7. Security Implications

### 7.1 Threat Mitigation

| Threat | Mitigation |
|---|---|
| Key compromise | `command=` restriction, short-lived certificates, rotation |
| Brute-force password | Lock password (`passwd -l`) |
| Interactive shell | nologin shell |
| Agent forwarding abuse | `no-agent-forwarding` |
| Tunnel abuse | `permitopen` and `no-port-forwarding` |

### 7.2 Defense-in-Depth

```bash
ssh-keygen -t ed25519 -f ~/.ssh/svc_key -C "svc-deploy@$(hostname)"
echo 'command="/usr/local/bin/backup.sh",no-agent-forwarding,no-port-forwarding,no-pty,no-X11-forwarding ssh-ed25519 AAAA...' \
    | sudo tee -a ~svc_deploy/.ssh/authorized_keys
sudo passwd -l svc_deploy && sudo usermod -s /sbin/nologin svc_deploy
# Optional Match block:
# Match User svc_deploy
#     ForceCommand /usr/local/bin/backup.sh
#     X11Forwarding no / AllowTcpForwarding no / PermitTTY no
# Monitor:
sudo journalctl -u sshd | grep "Accepted publickey" | grep svc_deploy
```

### 7.3 Logging

```bash
sudo journalctl -u sshd -f                              # Real-time
sudo journalctl -u sshd | grep "svc_deploy"             # Per-user
sudo tail -f /var/log/auth.log                          # Debian/Ubuntu
sudo tail -f /var/log/secure                            # RHEL/CentOS
# Log line: Accepted publickey for svc_deploy from 10.0.0.5:54321 ED25519 SHA256:abc...
```

---

## 8. Authorized Keys Options

Options are prepended to the public key line in `authorized_keys`:

```
command="/usr/local/bin/backup.sh",no-agent-forwarding,no-port-forwarding,no-pty,no-user-rc,no-X11-forwarding ssh-ed25519 AAAA... key-name
```

| Option | Effect |
|---|---|
| `command="cmd"` | Forces execution of `cmd` (overridden by `ForceCommand` in sshd_config) |
| `no-agent-forwarding` | Blocks SSH agent forwarding |
| `no-port-forwarding` | Blocks `-L`, `-R`, `-D` flags |
| `no-pty` | No PTY allocation (no interactive shell) |
| `no-X11-forwarding` | Blocks X11 forwarding |
| `permitopen="host:port"` | Tunnel-only to specific destinations |
| `from="pattern"` | Restrict to source IP/CIDR |
| `expiry-time="20261231"` | Key expires on a date |
| `environment="NAME=val"` | Set env var (requires `PermitUserEnvironment yes`) |

Multiple restricted keys in one file:

```
# ~svc_deploy/.ssh/authorized_keys
ssh-ed25519 AAAA... backup-master-key                                  # Full access
command="/usr/local/bin/monitor.sh",no-agent-forwarding,no-port-forwarding,no-pty ssh-ed25519 AAAA... monitor-key
no-agent-forwarding,no-X11-forwarding,permitopen="localhost:5432" ssh-ed25519 AAAA... tunnel-key
```

---

## 9. SSH Certificate-Based Authentication

```bash
# On CA host
ssh-keygen -t ed25519 -f /etc/ssh/ca_user_key -C "user-ca@example.com"
chmod 600 /etc/ssh/ca_user_key
ssh-keygen -s /etc/ssh/ca_user_key -I "svc_deploy-2026" -n "svc_deploy" -V "+52w" \
    -O source-address="10.0.0.0/8" -O force-command="/usr/local/bin/backup.sh" \
    -O no-agent-forwarding -O no-port-forwarding -O no-pty svc_deploy_key.pub

# On each SSH server
echo "TrustedUserCAKeys /etc/ssh/ca_user_key.pub" | sudo tee -a /etc/ssh/sshd_config
sudo cp ca_user_key.pub /etc/ssh/ca_user_key.pub && sudo systemctl restart sshd

# Client connects — no authorized_keys needed on server
ssh -i svc_deploy_key svc_deploy@server
```

**Benefits:** No per-server authorized_keys management, short-lived certificates with automatic expiry, embedded restrictions, central revocation via `RevokedKeys`, audit trail.

---

## 10. Troubleshooting

### 10.1 Diagnostic Checklist

```
☐ ~/.ssh mode 0700, owned by user?
☐ ~/.ssh/authorized_keys mode 0600, owned by user?
☐ Home directory NOT group/world writable?
☐ Key uncommented in authorized_keys?
☐ Private key matches public key?
☐ PubkeyAuthentication yes in sshd_config?
☐ User NOT in DenyUsers/DenyGroups?
☐ AllowUsers set? If so, user listed?
☐ Shell executable and in /etc/shells?
☐ Account not expired? (chage -l user)
☐ sshd restarted after config changes?
```

### 10.2 Using ssh -vvv

```bash
ssh -vvv -i ~/.ssh/test_key svc_deploy@server
# Authentications that can continue: publickey,password → key auth offered
# Offering public key: ... ED25519 SHA256:abc...        → client offering key
# Authentication succeeded (publickey).                  → key accepted
# Authentications that can continue: publickey           → key was rejected
```

### 10.3 Common Errors

| Error | Cause | Fix |
|---|---|---|
| `shell ... does not exist` | Path wrong or not in `/etc/shells` | Verify path, add to `/etc/shells` |
| `bad ownership or modes` | Wrong permissions | `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys` |
| `Permission denied (publickey)` | Key not accepted | `ssh -vvv`, check server logs |
| `account is locked` | PAM rejects | Check `chage -l user`, `usermod -U` |
| `Connection closed` immediately | nologin or ForceCommand | Check shell and forced commands |
| `no mutual signature algorithm` | RSA key too old | Use ed25519 keys |
| `This service allows sftp only` | ForceCommand internal-sftp | Use `sftp`, not `ssh` |

### 10.4 Isolation Tests

```bash
# Key auth only
ssh -o PreferredAuthentications=publickey -o PasswordAuthentication=no -i key user@host
# List offered methods
ssh -o PreferredAuthentications=none user@host
# Test shell
ssh -o RequestTTY=no user@host "echo hello"
# Test port forwarding
ssh -N -L 8080:localhost:80 user@host
# Verify config
sudo sshd -t
# Watch logs during a test
sudo journalctl -u sshd -f
# Check sshd listening
sudo ss -tlnp | grep :22
```

---

## 11. Quick-Reference Command Table

| Goal | Command |
|---|---|
| Create user with nologin | `sudo useradd -m -s /sbin/nologin user` |
| Change shell | `sudo usermod -s /sbin/nologin user` |
| Lock password | `sudo passwd -l user` |
| Unlock password | `sudo passwd -u user` |
| Check password status | `sudo passwd -S user` |
| Check account expiry | `sudo chage -l user` |
| Create ed25519 key | `ssh-keygen -t ed25519 -f ~/.ssh/key` |
| Install public key | `cat key.pub \| ssh u@h "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"` |
| Fix SSH perms | `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys` |
| Debug connection | `ssh -vvv -i key user@host` |
| Restart sshd | `sudo systemctl restart sshd` |
| Check config syntax | `sudo sshd -t` |
| View SSH logs | `sudo journalctl -u sshd -n 50` |
| SFTP connection | `sftp -i key user@host` |
| Tunnel only | `ssh -N -L 8080:localhost:80 user@host` |
| Add shell to list | `echo "/path/shell" \| sudo tee -a /etc/shells` |
| Sign key with CA | `ssh-keygen -s ca_key -I id -n user -V +52w key.pub` |

## References

- `man sshd_config`, `man sshd`, `man authorized_keys`, `man ssh-keygen`
- OpenSSH Project: https://www.openssh.com/
- SSH Academy: https://www.ssh.com/academy/ssh/authorized-keys-openssh
- DigitalOcean SSH Hardening: https://www.digitalocean.com/community/tutorials/how-to-harden-openssh-on-ubuntu-20-04

---

*Maintained at https://github.com/jackliusr/research*
