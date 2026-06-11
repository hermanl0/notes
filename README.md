# notes

My personal knowledge dump 
---

## Linux

### Filesystem Hierarchy Standard (FHS)

The **FHS** defines the standard layout of directories on Linux/Unix systems — what
lives where and why. It's maintained by the Linux Foundation, so the layout is
consistent across most distributions. Quick local references: `man hier` and `ls -l /`.

| Path | Purpose |
|------|---------|
| `/` | Root of the whole filesystem tree; everything hangs off it. |
| `/bin` | Essential user command binaries (e.g. `ls`, `cp`). On modern distros a symlink to `/usr/bin`. |
| `/sbin` | Essential system/admin binaries (e.g. `fdisk`, `ip`). Usually a symlink to `/usr/sbin`. |
| `/lib`, `/lib64` | Shared libraries needed by `/bin` and `/sbin`. Symlinked into `/usr/lib` on modern distros. |
| `/boot` | Boot loader files and the kernel (`vmlinuz`, `initrd`, GRUB config). |
| `/dev` | Device files — disks, terminals, etc. (e.g. `/dev/sda`, `/dev/null`). |
| `/etc` | System-wide configuration files (text). Host-specific, no binaries. |
| `/home` | Regular users' home directories (`/home/alice`). |
| `/root` | The root user's home directory (not `/home/root`). |
| `/opt` | Optional / third-party software installed as self-contained packages. |
| `/proc` | Virtual filesystem exposing process & kernel info (not on disk). |
| `/sys` | Virtual filesystem (sysfs) exposing kernel/device/driver info. |
| `/run` | Volatile runtime data since last boot (PIDs, sockets). A tmpfs, cleared on reboot. |
| `/srv` | Data served by the system (e.g. web, FTP content). |
| `/tmp` | Temporary files; often wiped on reboot. World-writable. |
| `/mnt` | Generic temporary mount point for manually mounting filesystems. |
| `/media` | Mount point for removable media (USB sticks, CDs) — usually auto-mounted. |
| `/usr` | Secondary hierarchy: the bulk of user programs & data (read-only, shareable). |
| `/var` | Variable data that changes at runtime (logs, caches, spools, mail). |

**`/usr` subdirectories** (the "shareable, read-only" hierarchy):

| Path | Purpose |
|------|---------|
| `/usr/bin` | Most user command binaries. |
| `/usr/sbin` | Non-essential system administration binaries. |
| `/usr/lib` | Libraries for `/usr/bin` and `/usr/sbin`. |
| `/usr/local` | Software compiled/installed locally by the admin (kept apart from distro packages). |
| `/usr/share` | Architecture-independent data (docs, man pages, icons). |
| `/usr/include` | C/C++ header files for development. |

**`/var` subdirectories** (variable / runtime data):

| Path | Purpose |
|------|---------|
| `/var/log` | System and application log files. |
| `/var/cache` | Application cache data (safe to delete; will be regenerated). |
| `/var/spool` | Queued data awaiting processing (print jobs, mail, cron). |
| `/var/lib` | State information for programs (databases, package manager state). |
| `/var/tmp` | Temporary files that should persist across reboots. |
| `/var/www` | Common (non-FHS-mandated) location for web server content. |

**Notes / gotchas**
- **The "/usr merge":** on modern distros `/bin`, `/sbin`, `/lib` are symlinks into `/usr`, so they're no longer separate directories — historical split from when `/usr` could be a separate disk.
- **Virtual filesystems:** `/proc` and `/sys` aren't stored on disk — the kernel generates their contents on the fly. `/run` and `/tmp` are typically `tmpfs` (RAM-backed).
- **Config vs data vs binaries:** rule of thumb — config in `/etc`, changing data in `/var`, programs in `/usr`.

**References:** `man hier` · [FHS 3.0 spec (Linux Foundation)](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)

### Finding the right command — `apropos`

When you know *what* you want to do but not the command's name, `apropos` searches
the short descriptions of every man page for a keyword (equivalent to `man -k`).

```bash
apropos <keyword>
```

Example:

```bash
$ apropos sudo
sudo (8)        - execute a command as another user
sudoers (5)     - default sudo security policy plugin
sudoreplay (8)  - replay sudo session logs
visudo (8)      - edit the sudoers file
```

The number in parentheses is the man section (1 = user commands, 5 = file formats,
8 = admin commands), so you can then run e.g. `man 5 sudoers`.

**Tip:** to decode a long or unfamiliar command, paste it into
[explainshell.com](https://explainshell.com/) — it breaks the command apart and
explains each flag.

### System enumeration / info-gathering

First things to run when landing on an unfamiliar Linux box — who am I, what is
this machine, what's it running, how is it networked. Most are pre-installed.
(Same skills used when reviewing a system's security posture.) Add `-h`/`--help`/`man`
to any of them for options.

**Identity & system**

| Command | What it tells you | Handy form |
|---------|-------------------|-----------|
| `whoami` | Current username | |
| `id` | Your UID/GID and group memberships | |
| `hostname` | The system's host name | |
| `uname` | OS / kernel / hardware info | `uname -a` (everything) |
| `pwd` | Current working directory | |
| `env` | Environment variables (or run a cmd in a modified env) | |

**Networking**

| Command | What it tells you | Handy form |
|---------|-------------------|-----------|
| `ip` | Interfaces, addresses, routes (modern) | `ip a`, `ip r` |
| `ifconfig` | Interface addresses/config (legacy, `net-tools`) | |
| `ss` | Open sockets / listening ports (modern) | `ss -tulpn` |
| `netstat` | Connections, routing, stats (legacy) | `netstat -tulpn` |

> Note: `ip` and `ss` are the modern replacements for the older `ifconfig` and
> `netstat` (which aren't installed by default on many newer distros).

**Processes & sessions**

| Command | What it tells you | Handy form |
|---------|-------------------|-----------|
| `ps` | Running processes | `ps aux` |
| `who` | Who is currently logged in | |

**Hardware & devices**

| Command | What it tells you |
|---------|-------------------|
| `lsblk` | Block devices (disks/partitions) as a tree |
| `lsusb` | Connected USB devices |
| `lspci` | PCI devices (GPU, NIC, controllers) |
| `lsof` | Open files — and the processes/sockets using them |

### Finding a file's inode number

Every file has an **inode** — a data structure holding metadata (permissions,
ownership, timestamps, size, block locations). The filename is just a pointer to
the inode. Two filenames can share an inode (hard links).

```bash
ls -i filename       # inode number only
ls -li               # long listing with inode numbers
stat filename        # full inode detail
```

Example:

```bash
$ ls -i /etc/hosts
131073 /etc/hosts

$ stat /etc/hosts
  File: /etc/hosts
  Size: 220       Blocks: 8    IO Block: 4096  regular file
Device: fd01h    Inode: 131073   Links: 1
```

### Sorting files by modification time

`ls -t` sorts by last-modified (newest first); pair with `-l` for the long listing.

```bash
ls -lt        # newest first
ls -ltr       # oldest first (reverse) — handy: newest ends up at the bottom, next to your prompt
ls -lt --time-style=long-iso   # explicit YYYY-MM-DD HH:MM timestamps
```

Related sort flags: `-S` (by size), `-X` (by extension), `-r` (reverse any sort), `-u` (use access time instead of modified time).
