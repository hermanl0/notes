# notes

My personal knowledge dump — commands, concepts, gotchas, and things I learn.
Unstructured on purpose: dump first, organize later. Anything that grows too big
gets split out into its own file/folder down the line.

---

## Kubernetes

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

## Networking

## Security / Blue team

## Azure / Cloud

## Tools

## TIL (today I learned)
