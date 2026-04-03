---
topic: "UniversalBlue Linux, Atomized Distros, Bazzite, SELinux, and POSIX File Permissions"
perspective: "Windows Systems Engineer, Intermediate"
date: 2026-04-03
staleness: 4
plugins:
  - context7
  - microsoft-docs
confidence: high
source_count: 67
tags:
  - ketchup
  - universal-blue
  - bazzite
  - selinux
  - posix-permissions
  - atomic-linux
  - immutable-os
  - rpm-ostree
  - fedora
  - windows-to-linux
---

# UniversalBlue Linux, Atomic Distros, Bazzite, SELinux, and POSIX Permissions: A Guide for Windows Systems Engineers

> [!abstract] TL;DR
> Atomic Linux (Bazzite/Universal Blue) delivers the OS as a signed container image — think WIM files pulled from a registry. The root filesystem is read-only, updates are staged and applied on reboot (never in-place), and rollback is a bootloader swap. SELinux adds mandatory access control on top of POSIX permissions — like if UAC and AppLocker had a baby that covered every file, socket, and process. POSIX permissions are simpler than NTFS ACLs (three slots, not a list), but special bits and SELinux contexts add layers you don't have on Windows.

## Why This Matters to You

You manage Windows systems. You know NTFS, GPO, DISM, Windows Update, and services.msc. Now you're looking at a Linux desktop — maybe Bazzite for gaming, maybe a work pilot — and the landscape has changed dramatically since 2022. Traditional Linux package management (`dnf install`, `apt install`) is being replaced by an image-based model that's actually closer to how you already think about Windows servicing than the old Linux way ever was.

The catch: the security model is completely different. POSIX permissions are structurally simpler than NTFS but behaviorally alien. SELinux adds a mandatory layer that has no direct Windows equivalent. And the immutable/atomic OS model means your instincts about "just install it" and "just chmod it" will lead you astray.

This report bridges every concept from what you already know.

## What This Is NOT

> [!warning] Wrong mental models to discard before reading further

- **"Immutable" does not mean "frozen."** You can still install software, change configs, and customize everything. The *mechanism* is different (layering, Flatpak, containers), not the capability.
- **SELinux is not a firewall.** It doesn't filter network traffic. It controls what processes can do to files, sockets, and other processes — even root.
- **SELinux is not Windows Defender.** It's not antimalware. It's mandatory access control — a policy engine that says "this process type may only access these file types."
- **POSIX permissions are not "simplified NTFS."** They're a fundamentally different model with different constraints. You cannot bolt on per-user exceptions the way you add NTFS ACEs.
- **Bazzite is not "Linux Mint for gamers."** It's an atomic, image-based OS with a completely different update and package model from traditional distros.

---

## Core Concepts: Atomic/Immutable Linux

### The Problem This Solves

In 2022, if a Windows Update partially applied and broke a DLL dependency, you could end up in a hybrid state that's neither the old system nor the new one. Traditional Linux package managers (`apt`, `dnf`) share the same flaw — they mutate a running system file-by-file. Atomic Linux eliminates that entire failure class.

### What "Immutable" Means in Practice

The OS vendor-owned filesystem is read-only, and all mutations happen in a staging area that only takes effect at reboot:

- **`/usr` is mounted read-only** — all OS binaries, libraries, system files. You cannot write to it at runtime, even as root. Think `C:\Windows\System32` protected by something harder than TrustedInstaller ([OSTree Overview](https://ostreedev.github.io/ostree/introduction/)).
- **`/etc` is writable and versioned** — config changes survive upgrades via a 3-way merge, similar to how GPO merges apply over a base template ([OSTree Overview](https://ostreedev.github.io/ostree/introduction/)).
- **`/var` is writable and shared** — user data, logs, application state. Persists across all deployments unmodified ([OSTree Overview](https://ostreedev.github.io/ostree/introduction/)).

### The Engine: OSTree (Git for OS Binaries)

The technology underneath is **OSTree** — a content-addressed object store for bootable filesystem trees. If you've used Git, the mental model maps directly:

| OSTree | Git | Windows |
|--------|-----|---------|
| Object store in `/ostree/repo` | `.git/objects` | Component Store (`WinSxS`) |
| Deployment (checked-out bootable tree) | Working tree at a commit | A mounted WIM image |
| Commit (SHA256 filesystem snapshot) | Git commit | A WIM snapshot |
| Hardlinks between deployments | Pack deduplication | WinSxS hard-linking |
| `rpm-ostree upgrade` | `git pull` + checkout | Apply pending Windows Update |

Each deployment lives at `/ostree/deploy/$name/$checksum` and is built from **hardlinks** into the object store — keeping two OS versions on disk costs only the delta ([OSTree content-addressed store](https://ostreedev.github.io/ostree/reference/ostree-OstreeRepo.html)) _(~inferred: same deduplication principle as WinSxS hard-linking)_.

### How Updates Are Staged

When you run `rpm-ostree upgrade`:

1. New OS image is **downloaded in the background** — your running system is untouched
2. Bootloader entries are updated atomically (symlink swap) ([OSTree atomic upgrades](https://ostreedev.github.io/ostree/atomic-upgrades/))
3. Your `/etc` changes are 3-way merged into the new deployment
4. You reboot. New OS is live. Old deployment is preserved on disk.

> [!info] DISM Analogy
> This is closest to `DISM /Apply-Image` to a new offline path, updating BCD to point at it, keeping the old installation intact — except OSTree does it automatically, atomically, and with deduplication.

### The OCI Image Layer (New Since 2022)

**This did not exist in its current form in 2022.** Universal Blue and Bazzite distribute the OS as **OCI container images** — the same format as Docker/Podman images — hosted on `ghcr.io` ([Universal Blue GitHub](https://github.com/ublue-os)). The bridge technology is **bootc** (Bootable Containers), which reached stable status in 2024 ([bootc getting started](https://developers.redhat.com/articles/2024/09/24/bootc-getting-started-bootable-containers)).

| OCI/bootc | Windows |
|-----------|---------|
| OCI image on `ghcr.io` | WIM/ISO on a CDN |
| `rpm-ostree rebase ostree-image-signed:docker://ghcr.io/ublue-os/bazzite:latest` | `DISM /Apply-WIM` pointed at a new image |
| `cosign` signature verification | Authenticode signature on WIMs |
| `latest` channel | Windows Current Branch |
| `gts` channel | Windows LTSC ([Universal Blue forums](https://universal-blue.discourse.group/t/what-is-the-path-for-promoting-latest-to-stable-to-gts/4376)) |

### Rollback: Better Than System Restore

OSTree always keeps at least two deployments on disk. Rolling back is a bootloader operation:

```bash
rpm-ostree rollback
```

The previous deployment was **never removed**. On next reboot, you're running the old OS. You can also select it from the GRUB boot menu — like "Last Known Good Configuration" except it works reliably and atomically ([Bazzite rollback docs](https://universal-blue.discourse.group/t/rolling-back-system-updates-on-bazzite/2644)).

| Mechanism | Scope | Reliability |
|-----------|-------|-------------|
| Windows System Restore | Registry + some files | Partial |
| WinRE Recovery | Full OS reinstall | Reliable but destructive |
| OSTree rollback | Complete filesystem swap | Atomic, non-destructive, always available |

---

## The UniversalBlue & Bazzite Ecosystem

### The SKU Hierarchy

| Windows | Linux Equivalent |
|---------|-----------------|
| Windows Core (bare OS) | Fedora Atomic Desktop (Silverblue/Kinoite) |
| Windows Enterprise (hardened base) | Universal Blue `main` images |
| Windows Gaming/specialized SKU | Bazzite |

**Universal Blue** ([universal-blue.org](https://universal-blue.org/)) is a community project that publishes custom Fedora Atomic images using standard OCI container tooling. Their own framing: "not a distribution, but a new way to consume existing distributions." The build pipeline is GitHub Actions running `podman build`, producing signed OCI images pushed to `ghcr.io/ublue-os/`.

**Bazzite** ([bazzite.gg](https://bazzite.gg/)) is the gaming-and-desktop flagship — pre-tuned drivers, Steam, Proton, HDR patches, Steam Deck support. Ships KDE Plasma and GNOME variants. Includes a custom kernel with gaming patches (fsync, HDR) baked into the signed image — no "install the gaming kernel" step ([Bazzite FAQ](https://docs.bazzite.gg/General/FAQ/)).

> [!tip] Change Since 2022
> In 2022, atomic Linux was a niche experiment. As of 2024-2026, it is Fedora's recommended desktop path and the OCI-image delivery model is stable infrastructure _(~inferred: based on Fedora's OstreeNativeContainerStable change and Universal Blue's Fedora 42 announcement)_.

### How Updates Work

**Windows Update** patches the running OS in-place. **Bazzite** pulls a new complete OS image alongside the running deployment and sets it as default for next boot. The running system is untouched until reboot ([Bazzite Update Guide](https://github.com/bazzite-org/docs.bazzite.gg/blob/main/src/Installing_and_Managing_Software/Updates_Rollbacks_and_Rebasing/updating_guide.md)).

Manual trigger:

```bash
ujust update
```

This runs Topgrade under the hood — updates base image, Flatpaks, Distrobox containers, and Homebrew in one operation.

### ujust: Your GPO Script Library

`ujust` wraps the `just` task runner with pre-built system configuration recipes shipped as part of the OS image ([Bazzite ujust docs](https://docs.bazzite.gg/Installing_and_Managing_Software/ujust/)):

| Command | Windows Equivalent |
|---------|--------------------|
| `ujust update` | `wuauclt /detectnow` + app updates |
| `ujust clean-system` | Disk Cleanup + DISM component cleanup |
| `ujust setup-virtualization` | Enable Hyper-V + IOMMU policy |
| `ujust enable-tailscale` | Deploy VPN client via GPO |

Run `ujust --choose` for an interactive menu.

### Software Management: Three Tiers

| Tier | Tool | Windows Equivalent | When to Use |
|------|------|--------------------|-------------|
| 1 (default) | **Flatpak** | Microsoft Store / winget | GUI apps — sandboxed, independent of OS |
| 2 (dev tools) | **Distrobox** | WSL2 | Full package management inside a container |
| 3 (last resort) | **rpm-ostree install** | DISM /Add-Package | System-level components only — requires reboot |

```bash
# Tier 1: Install an app via Flatpak
flatpak install flathub com.visualstudio.code

# Tier 2: Create an Ubuntu container for dev tools
distrobox create --name toolbox --image ubuntu:24.04
distrobox enter toolbox

# Tier 3: Layer a system package (requires reboot)
rpm-ostree install fuse-sshfs
```

> [!warning] Minimize rpm-ostree layering
> The more you layer, the more complex upgrades become. Bazzite encourages Flatpak and Distrobox to keep the base image clean ([Bazzite rpm-ostree docs](https://docs.bazzite.gg/Installing_and_Managing_Software/rpm-ostree/)).

---

## SELinux for Windows Admins

### The Mental Model Shift: DAC to MAC

You already live in a DAC (Discretionary Access Control) world. NTFS permissions are DAC: the owner decides who gets access. Windows bolted on Mandatory Integrity Control (MIC) post-Vista — integrity levels (Low, Medium, High, System) stored in the SACL — but it's shallow: it governs write-up/write-down, not fine-grained cross-resource interaction ([Microsoft Docs — MIC](https://learn.microsoft.com/windows/win32/secauthz/mandatory-integrity-control)).

SELinux is a full MAC system. The kernel enforces access rules that *no process can override* — not even root. The policy defines what is allowed; everything else is denied. Think of it as if AppLocker's "default deny" applied to every file open, every socket, every IPC call — and UAC prompts couldn't bypass it.

### Security Contexts: SELinux SIDs

Every process and file carries a **security context** label: `user:role:type:level`. For the targeted policy (default on Fedora/Bazzite), the **type** field does almost all the work ([Red Hat Docs](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/chap-security-enhanced_linux-selinux_contexts)):

```bash
ls -Z /var/www/html/index.html
# system_u:object_r:httpd_sys_content_t:s0

ps -eZ | grep httpd
# system_u:system_r:httpd_t:s0
```

The policy says: processes in `httpd_t` may read files of type `httpd_sys_content_t`. The type label is the SID, the domain is the token, the policy rules are the ACL entries — except written by the OS vendor, not you, and compiled into a binary loaded at boot.

### Targeted Policy: Default-Allow with Selective Confinement

Regular users run in `unconfined_t` — effectively open DAC-fallback behavior ([Red Hat Docs](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/sect-security-enhanced_linux-targeted_policy-unconfined_processes)). Only high-risk daemons (Apache, Nginx, sshd, named) are tightly confined.

Think of Windows Firewall's default-allow-outbound posture with specific application-layer rules blocking named services from doing things they shouldn't.

### Booleans: The GPO Settings of SELinux

Pre-written policy toggles for common adjustments ([Red Hat Docs](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/sect-security-enhanced_linux-working_with_selinux-booleans)):

```bash
# Check if Apache can make outbound network connections
getsebool httpd_can_network_connect
# httpd_can_network_connect --> off

# Enable persistently (-P = survives reboot, like applying a GPO)
setsebool -P httpd_can_network_connect on

# List all booleans for a service
getsebool -a | grep httpd
```

> [!danger] Always use `-P` for persistent changes
> Without `-P`, the boolean resets on reboot — like a GPO setting that reverts to "Not Configured."

### The #1 Gotcha: File Contexts Don't Follow Copies

You copy a config file from `~` into `/etc/nginx/conf.d/`. It carries its source label (`user_home_t`). Nginx can't read it — the policy requires `httpd_config_t` ([Red Hat Docs](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/sect-security-enhanced_linux-troubleshooting-top_three_causes_of_problems)):

```bash
ls -Z /etc/nginx/conf.d/myapp.conf
# unconfined_u:object_r:user_home_t:s0  ← WRONG

# Fix permanently (survives restorecon):
semanage fcontext -a -t httpd_config_t "/etc/nginx/conf.d(/.*)?"
restorecon -Rv /etc/nginx/conf.d/
```

> [!warning] Do NOT use `chcon` as a permanent fix
> `chcon` sets the label directly but gets wiped on the next `restorecon` or relabel — like editing a local policy that domain GPO overrides on refresh. Use `semanage fcontext` + `restorecon`. That pair is permanent ([Red Hat Docs](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/sect-security-enhanced_linux-working_with_selinux-selinux_contexts_labeling_files)).

### Troubleshooting: Event Viewer → audit.log

| Windows | SELinux |
|---------|---------|
| Event Viewer | `/var/log/audit/audit.log` |
| Filter by source | `ausearch -m AVC -ts recent` |
| "What rule blocked this?" | `audit2why -a` |
| "Give me an allow rule" | `audit2allow -a` |
| Apply the rule | `semodule -i mypolicy.pp` |

```bash
# 1. Find recent denials
ausearch -m AVC,USER_AVC -ts recent

# 2. Human-readable explanation
audit2why -a

# 3. Generate and apply a policy module (REVIEW FIRST)
audit2allow -a -M mypolicy
semodule -i mypolicy.pp
```

> [!warning] Treat `audit2allow` output like a firewall "allow all" rule
> It will work — but read what it's actually allowing before you load it. A broad generated rule can silently open more than you intended ([Red Hat RHEL 9 SELinux Guide](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/using_selinux/troubleshooting-problems-related-to-selinux_using-selinux)).

**Most common denial causes** ([Red Hat Docs](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/sect-security-enhanced_linux-troubleshooting-top_three_causes_of_problems)):
1. Wrong file context label (copy/move or non-standard path)
2. Boolean not enabled
3. Missing port label (`semanage port -a -t http_port_t -p tcp 8443`)
4. Legitimate policy gap (write a module)

### "Just Disable SELinux" = "Just Disable UAC"

Same energy, same risk. Re-enabling after full disable requires a filesystem relabel at next boot. On Bazzite (atomic/immutable), this is especially disruptive _(~inferred: aggregate likelihood from atomic distro architecture; relabel behavior on OSTree-based systems needs verification)_.

Use `setenforce 0` (Permissive mode) to diagnose — it logs what *would* be denied without blocking. Fix the root cause, then go back to Enforcing ([CentOS Blog](https://blog.centos.org/2017/07/dont-turn-off-selinux/); [TechRepublic](https://www.techrepublic.com/article/why-its-time-to-stop-setting-selinux-to-permissive-or-disabled/)).

---

## POSIX File Permissions Deep Dive

### The Core Model: Three Slots, Not an ACL

NTFS uses a DACL — a list of ACEs, each naming a specific principal. POSIX permissions use exactly **three permission slots** for three classes:

| POSIX Class | Windows Equivalent |
|-------------|-------------------|
| **user** (owner) | File Owner in the DACL |
| **group** | One named group in the DACL |
| **other** | "Everyone" / "Authenticated Users" |

The key constraint: only **one group** at a time. For multiple groups with different rights, you need POSIX ACLs ([O'Reilly — POSIX vs NTFS](https://www.oreilly.com/library/view/os-x-mountain/9781118417812/a6_14_9781118417812-ch08.html)).

> [!danger] Do not import the NTFS ACL mental model
> On NTFS you routinely add per-user ACEs. On bare rwx you cannot. The three-slot model is a hard constraint.

### Octal Notation → NTFS Rights

Each slot encodes **r=4, w=2, x=1**, summed per class:

| Octal | rwx | NTFS Equivalent |
|-------|-----|-----------------|
| `7` | `rwx` | Full Control |
| `6` | `rw-` | Modify |
| `5` | `r-x` | Read & Execute |
| `4` | `r--` | Read |
| `0` | `---` | Deny All |

`chmod 755` = Owner Full Control, Group/Everyone Read & Execute. `chmod 644` = Owner Modify, Group/Everyone Read. These are the two most common defaults ([SSD Nodes](https://www.ssdnodes.com/blog/linux-permissions-chmod-755-644-drwxrxrx-explained/)).

> [!info] Execute on directories ≠ "run this folder"
> The `x` bit on a **directory** is the traverse/enter bit. Without it, you cannot `cd` into the directory even if you can read its listing. This is separate from "List folder contents" in NTFS terms ([Red Hat](https://www.redhat.com/en/blog/linux-file-permissions-explained)).

### chmod, chown, chgrp: The Security Tab in Terminal

| Command | What it does | Windows equivalent |
|---------|-------------|-------------------|
| `chmod 755 /path` | Set permission bits | Edit ACEs in Security tab |
| `chown alice:devs /path` | Change owner + group | "Change owner" in Advanced Security |
| `chgrp finance /path` | Change group only | Edit group principal in DACL |

Only `root` can reassign ownership to another user ([DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-permissions-linux)). Recursive: `chmod -R 755 /srv/app/` (like "Replace all child object permissions") _(unsourced)_.

### umask: Default Permission Template

NTFS new files inherit ACEs from parent folder inheritance rules. Linux uses `umask` — a mask of bits to **remove** from the system default:

- System starts with `666` for files, `777` for directories
- Default umask `022` produces: files `644`, directories `755`

([nixCraft](https://www.cyberciti.biz/tips/understanding-linux-unix-umask-value-usage.html))

In WSL, configure via `/etc/wsl.conf`:

```ini
[automount]
options = "metadata,umask=22,fmask=11"
```

Without `metadata` enabled, `chmod` only sets the Windows read-only attribute — Linux permission bits don't persist on NTFS ([Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/file-permissions)).

### POSIX ACLs: When Three Slots Aren't Enough

When you need Finance=Read and Marketing=Read+Write on the same file, `setfacl`/`getfacl` bring per-principal entries closer to NTFS:

```bash
setfacl -m g:finance:r-- /srv/reports/q1.csv
setfacl -m g:marketing:rw- /srv/reports/q1.csv
getfacl /srv/reports/q1.csv
```

Supported on ext4, XFS, Btrfs ([Arch Wiki](https://wiki.archlinux.org/title/Access_Control_Lists)).

> [!warning] POSIX ACLs cannot DENY
> NTFS has both Allow and Deny ACEs. POSIX ACLs are **additive only**. For deny semantics, restructure ownership or use SELinux ([NTFS.com](https://www.ntfs.com/ntfs-permissions-acl-use.htm)).

### Special Bits: No Windows Equivalent

**Setuid** (`s` in owner execute slot): When an executable with setuid runs, it executes as the **file owner** regardless of who launched it. This is how `passwd` writes to `/etc/shadow` — it runs as root even when launched by `jsmith` ([Red Hat](https://www.redhat.com/en/blog/suid-sgid-sticky-bit)):

```
-rwsr-xr-x root root /usr/bin/passwd
     ^--- setuid bit
```

**Setgid on a directory**: New files inherit the directory's group, not the creator's primary group. Critical for shared project directories ([GeeksforGeeks](https://www.geeksforgeeks.org/linux-unix/setuid-setgid-and-sticky-bits-in-linux-file-permissions/)):

```bash
chmod g+s /srv/shared/team-project/
```

**Sticky bit on a directory**: Prevents users from deleting files they don't own, even with write permission on the directory. `/tmp` uses this — everyone can write, but only delete their own files _(~inferred: no direct NTFS equivalent; would need per-file ACEs)_:

```
drwxrwxrwt  /tmp
          ^--- sticky bit
```

### root vs Administrator

Windows Administrators is a **group** — multiple accounts can be members. Linux `root` is a **single account** (UID 0). `sudo` executes a specific command as root and drops back — no persistent elevated session like Windows' split token ([BeyondTrust](https://www.beyondtrust.com/blog/entry/unix-linux-privileged-management-should-you-sudo)).

> [!tip] Since 2022: Windows got sudo too
> Windows 11 24H2 (2024) shipped a native `sudo` command — but it's all-or-nothing elevation, not per-command configurable like Linux's `/etc/sudoers` ([Petri](https://petri.com/windows-sudo/)).

---

## Quick Reference

### Essential Commands

| Task | Command | Windows Equivalent |
|------|---------|-------------------|
| Check OS status | `rpm-ostree status` | `winver` / `Get-ComputerInfo` |
| Update OS | `rpm-ostree upgrade` | Windows Update |
| Rollback | `rpm-ostree rollback` | Last Known Good Config |
| Install app | `flatpak install flathub <app>` | `winget install <app>` |
| Layer system package | `rpm-ostree install <pkg>` | `DISM /Add-Package` |
| Check logs | `journalctl -xe` | Event Viewer (Errors) |
| Service status | `systemctl status <svc>` | `sc query <svc>` |
| Start/stop service | `systemctl start/stop <svc>` | `sc start/stop <svc>` |
| Enable at boot | `systemctl enable <svc>` | Set startup = Automatic |
| Check firewall | `firewall-cmd --get-active-zones` | `netsh advfirewall show` |
| Open port | `firewall-cmd --zone=public --add-port=8080/tcp --permanent && firewall-cmd --reload` | WFAS inbound rule |
| SELinux status | `sestatus` / `getenforce` | N/A |
| SELinux file context | `ls -Z /path` | N/A |
| SELinux denials | `ausearch -m AVC -ts recent` | N/A |
| File permissions | `ls -la /path` | Properties > Security > Effective Access |
| Change permissions | `chmod 755 /path` | Edit ACEs |
| Change owner | `chown user:group /path` | Advanced Security > Change owner |
| System config recipes | `ujust --choose` | N/A (closest: PowerShell DSC) |

### SELinux Troubleshooting Cheat Sheet

```bash
# Is SELinux blocking something?
getenforce                          # Should say "Enforcing"
ausearch -m AVC -ts recent          # Recent denials

# What's the file labeled?
ls -Z /path/to/problem/file

# Fix wrong label permanently
semanage fcontext -a -t <correct_type> "/path(/.*)?"
restorecon -Rv /path/

# Toggle a boolean
getsebool -a | grep <service>
setsebool -P <boolean_name> on

# Custom port
semanage port -a -t <port_type> -p tcp <port>

# Nuclear option (diagnosis only, NOT production)
setenforce 0                        # Permissive — logs but doesn't block
```

---

## Gotchas & Pitfalls

> [!danger] Things that WILL trip you up coming from Windows

1. **"Permission denied" has two independent layers.** POSIX permissions can say "allowed" while SELinux says "denied." Always check both: `ls -la` AND `ls -Z`. This is like having NTFS ACLs and AppLocker both active — either can block you.

2. **Copying files changes nothing about their SELinux labels.** `cp` from your home dir into a service directory carries the wrong label. Always `restorecon` after placing files in non-standard locations.

3. **`rpm-ostree install` requires a reboot.** There is no "install and use immediately." This is by design — the atomic model stages all changes for next boot.

4. **Flatpak apps are sandboxed by default.** If an app can't see your files, check its permissions with Flatseal. This is like Windows Store apps needing explicit file system access capability.

5. **`sudo` is not "Run as Administrator."** It runs one command as root and drops back. There is no persistent elevated shell unless you explicitly create one with `sudo -i` (and you shouldn't).

6. **firewall-cmd without `--permanent` is temporary.** Rules vanish on reload/reboot. Always pair with `--permanent` and `--reload`.

7. **umask is not inheritance.** It's a subtraction mask applied at file creation time. Existing files are not retroactively affected. NTFS inheritance propagates to existing children; umask does not.

8. **POSIX ACLs cannot deny.** If you need to explicitly block a user from a file, you must restructure ownership or use SELinux — there is no "Deny" ACE equivalent.

9. **Don't disable SELinux.** Use Permissive mode (`setenforce 0`) to diagnose, then fix the root cause and go back to Enforcing. Disabling fully requires a filesystem relabel to re-enable.

10. **The `gts` channel is not identical to `latest`.** GTS tracks Fedora N-1 and updates less frequently — like LTSC vs Current Branch. Know which channel you're on: `rpm-ostree status`.

---

## Further Reading

**Atomic Linux / Universal Blue:**
- [OSTree Documentation](https://ostreedev.github.io/ostree/introduction/) — The engine underneath everything
- [rpm-ostree Administrator Handbook](https://coreos.github.io/rpm-ostree/administrator-handbook/) — Day-to-day operations reference
- [Universal Blue](https://universal-blue.org/) — Project homepage and philosophy
- [Bazzite Documentation](https://docs.bazzite.gg/) — Installation, updates, troubleshooting
- [bootc Getting Started](https://developers.redhat.com/articles/2024/09/24/bootc-getting-started-bootable-containers) — The future of image-based Linux

**SELinux:**
- [The SELinux Notebook](https://github.com/selinuxproject/selinux-notebook) — Comprehensive reference (Context7)
- [Red Hat SELinux Guide (RHEL 9)](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/using_selinux/troubleshooting-problems-related-to-selinux_using-selinux) — Current troubleshooting guide
- [End Point Dev — Streamlining SELinux Policies (2024)](https://www.endpointdev.com/blog/2024/08/streamlining-selinux-policies/) — Writing custom modules

**POSIX Permissions:**
- [Microsoft Learn — WSL File Permissions](https://learn.microsoft.com/en-us/windows/wsl/file-permissions) — How NTFS and POSIX interact in WSL
- [Red Hat — Linux File Permissions Explained](https://www.redhat.com/en/blog/linux-file-permissions-explained) — Fundamentals with examples
- [Arch Wiki — Access Control Lists](https://wiki.archlinux.org/title/Access_Control_Lists) — POSIX ACL reference

**Daily Operations:**
- [DigitalOcean — journalctl Guide](https://www.digitalocean.com/community/tutorials/how-to-use-journalctl-to-view-and-manipulate-systemd-logs) — Log analysis
- [Distrobox](https://distrobox.it/) — Container-based traditional package management
- [Red Hat — Beginner's Guide to firewalld](https://www.redhat.com/en/blog/beginners-guide-firewalld) — Zone-based firewall management
