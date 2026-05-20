# Proxmox APT Update Fix (401 Unauthorized / Exit Code 100)

Fixes:

- `TASK ERROR: command 'apt-get update' failed: exit code 100`
- `401 Unauthorized`
- `repository is not signed`
- Enterprise repository issues
- Duplicate Debian repository warnings

---

# Why This Error Happens

By default, Proxmox enables the **enterprise repositories**:

- `enterprise.proxmox.com/debian/pve`
- `enterprise.proxmox.com/debian/ceph-*`

These repositories require a **paid Proxmox subscription**.

If you do not have a subscription, running:

```bash
apt update
```

will fail with:

```bash
401 Unauthorized
```

and:

```bash
TASK ERROR: command 'apt-get update' failed: exit code 100
```

Additionally, newer Proxmox versions use `.sources` files instead of `.list`, so simply commenting old `.list` files may NOT fix the issue.

---

# One Command Fix

Run this command as root:

```bash
mv /etc/apt/sources.list.d/pve-enterprise.sources /root/ 2>/dev/null && \
mv /etc/apt/sources.list.d/ceph.sources /root/ 2>/dev/null && \
mv /etc/apt/sources.list.d/debian.sources /root/ 2>/dev/null && \
echo "deb http://deb.debian.org/debian trixie main contrib non-free-firmware
deb http://deb.debian.org/debian trixie-updates main contrib non-free-firmware
deb http://security.debian.org/debian-security trixie-security main contrib non-free-firmware
deb http://download.proxmox.com/debian/pve trixie pve-no-subscription
deb http://download.proxmox.com/debian/ceph-squid trixie no-subscription" > /etc/apt/sources.list && \
apt clean && apt update
```

---

# Upgrade System

After fixing repositories:

```bash
apt full-upgrade -y
reboot
```

---

# What This Command Does

- Disables enterprise repositories
- Removes duplicate Debian source definitions
- Adds free Proxmox no-subscription repositories
- Cleans APT cache
- Updates package lists

---

# Tested On

- Proxmox VE 9
- Debian Trixie
- Ceph Squid

---

# Verify Enterprise Repositories

To check if enterprise repositories are still enabled:

```bash
grep -R "enterprise.proxmox.com" /etc/apt/
```

---

# License

MIT License
