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

### Finding files with `find`

`find` walks a directory tree and filters by name, type, size, time, owner,
permissions and more. Tests are combined with an implicit AND. General shape:

```bash
find <where> <tests> <action>
```

Common tests:

| Test | Matches |
|------|---------|
| `-type f` / `-type d` | Regular files / directories |
| `-name "*.conf"` | Name by glob (`-iname` = case-insensitive) |
| `-size +10k` / `-size -1M` | Larger than 10 KiB / smaller than 1 MiB (`k`, `M`, `G`; `c` = bytes) |
| `-newermt "2023-01-01"` | Modified after a date/time (`m`time, `t`imestamp) |
| `-mtime -7` / `-mtime +30` | Modified within the last 7 days / more than 30 days ago |
| `-user <name>` / `-group <name>` | Owned by user / group |
| `-perm 644` / `-perm -u+s` | Exact mode / has at least these bits (e.g. SUID) |

Stack tests to narrow results — e.g. a `.conf` file within a size range, modified after a date:

```bash
find / -type f -name "*.conf" -newermt "2023-01-01" -size +10k -size -1M 2>/dev/null
```

**Notes / gotchas**
- Tests are ANDed by default; use `-o` for OR and `!` / `-not` to negate.
- `-size +10k` is *strictly greater than* 10×1024 bytes, `-size -1M` *strictly less than* 1×1024×1024 — the pair brackets an open range.
- `find` rounds sizes **up** to the next unit, so `-size 1k` matches 1–1024 bytes. Use the `c` suffix when you need exact byte counts.
- There's no portable "creation time" test — `-newermt` / `-mtime` use **modification** time.
- `2>/dev/null` hides the `Permission denied` noise when searching from `/` as a normal user.
- Act on matches with `-exec`, e.g. `find . -name '*.log' -exec rm {} \;` (or the built-in `-delete`).

### Locating a command — `which`, `whereis`, `type`

To find *where* a command lives on disk (and what the shell actually runs):

```bash
which python3        # first match in $PATH        -> /usr/bin/python3
type python3         # how the shell resolves it (builtin/alias/function/file)
command -v python3   # POSIX-portable equivalent of `which`
whereis python3      # binary + source + man-page locations
```

| Tool | Shows |
|------|-------|
| `which <cmd>` | Path of the executable found first in `$PATH` |
| `type <cmd>` | Whether it's a shell builtin, alias, function or file (plus the path) |
| `command -v <cmd>` | Same as `which`, but built into the shell — portable in scripts |
| `whereis <cmd>` | Binary, source, and man-page paths (searches standard dirs, not `$PATH`) |

**Notes / gotchas**
- `which` only searches `$PATH`; `whereis` searches a fixed set of standard directories, so their results can differ.
- Aliases and shell builtins have no file path — use `type` to see what actually runs before assuming a binary exists.

### Listing installed packages (Debian / `apt`)

Debian-based systems (Debian, Ubuntu, Kali, Parrot) manage software with `dpkg`
(the low-level package database) and `apt` (the high-level front end).

```bash
dpkg -l                    # list all packages (status in the first column)
dpkg -l | grep -c "^ii"    # count installed packages (ii = installed OK)
apt list --installed       # apt's view of installed packages
dpkg -S /bin/ls            # which package owns a given file
dpkg -L <package>          # list every file a package installed
```

The status column of `dpkg -l`: `ii` = installed, `rc` = removed but config files remain.

**Other distros:** on RPM-based systems (RHEL, Fedora, Rocky) the equivalents are
`rpm -qa` / `dnf list installed` (count with `rpm -qa | wc -l`).

### Listening sockets & bind addresses

`ss -tulpn` (or legacy `netstat -tulpn`) lists listening sockets:
`-t` TCP, `-u` UDP, `-l` listening, `-p` owning process, `-n` numeric ports.

What the **Local Address** column tells you — who can actually reach the service:

| Local address | Reachable from |
|---------------|----------------|
| `0.0.0.0:<port>` | All IPv4 interfaces (LAN + internet-facing) |
| `[::]:<port>` (or `*:<port>`) | All IPv6 interfaces / dual-stack socket |
| `127.0.0.1` / `127.0.0.53%lo` | Localhost only — not reachable off-box |
| `<specific IP>:<port>` | Only that one interface/address |

```bash
ss -tulpn                     # everything listening
ss -tuln -4                   # IPv4 only  (-6 for IPv6)
# count listeners bound to all IPv4 interfaces:
ss -tuln -4 -H | awk '{print $5}' | grep -c '^0\.0\.0\.0:'
```

**Notes / gotchas**
- Don't `grep 0.0.0.0` over the whole line — the **Peer Address** column is `0.0.0.0:*` for *every* listener, so it also matches localhost-only services. Filter on the **Local Address** column instead.
- UDP sockets show state `UNCONN`, not `LISTEN` — they're still bound and serving, so decide whether "listening" should include them.
- `*:<port>` is an IPv6 (dual-stack) socket — exclude it when you specifically want IPv4.

### Which user a process runs as

The first column of `ps aux` is the **owning user** — the privilege context a
service runs under (handy for spotting daemons running as `root` that shouldn't).

```bash
ps aux | grep -i "[n]ginx"      # USER column shows the account, e.g. 'www-data'
ps -o user= -C nginx            # just the owner of a named process, no grep
```

**Notes / gotchas**
- The `[n]ginx` bracket trick stops the pattern matching its own `grep` process — the literal string `[n]ginx` never contains `nginx`, so no spurious grep line.
- Other ways in: `pgrep -a nginx`, or `systemctl show -p User <service>` for the configured service user.

### Fetching a page and extracting links (`curl` + `grep`)

`curl -s <url>` prints a page's source; pipe it through `grep -oP` to pull out
just the parts you want. Extract every unique link to a domain:

```bash
curl -s https://example.com \
  | grep -oP 'https?://[^"]+' \   # -o: print only matches, -P: Perl regex
  | grep 'example\.com' \         # keep same-domain links only
  | sort -u                       # unique  (add: | wc -l  to count)
```

Handy `curl` flags: `-s` silent, `-L` follow redirects, `-I` headers only,
`-o <file>` save output, `-A '<ua>'` set User-Agent.

**Notes / gotchas**
- `grep -oP` prints only the matched text, so you get clean tokens out of noisy HTML instead of whole lines.
- URLs that differ only by a `?query` string count as **distinct** — strip them with `sed 's/?.*//'` first if you want paths only.
- A loose domain pattern can match URL-encoded strings inside query params (e.g. `%2Fwww.example.com`), inflating counts — anchor the pattern to `https?://` and the exact host.
- `sort -u` = `sort | uniq`; append `| wc -l` to count.
- `curl` prints to stdout (use `-o`/`-O` to save); `wget <url>` downloads to a file by default (e.g. `index.html`) — handy as a quick download manager.

### Regular expressions with `grep`

`grep` matches lines against a pattern. Basic regex (BRE) is the default; `-E`
enables extended regex (ERE), so `()`, `{}`, `|`, `+`, `?` work without backslashes.

Building blocks:

| Pattern | Matches |
|---------|---------|
| `.` | Any single character |
| `.*` | Any run of characters — between two terms it acts like an ordered AND |
| `[a-z]` | Character class — any one of the listed characters/ranges |
| `(foo)` | Group — treat the enclosed pattern as a unit |
| `a\|b` | OR — either `a` or `b` (needs `-E`) |
| `x{1,3}` | Quantifier — repeat the previous item 1–3 times (needs `-E`) |
| `^foo` | Line **starts** with `foo` |
| `foo$` | Line **ends** with `foo` |
| `\bfoo` / `foo\b` | Word boundary — word starting / ending with `foo` |

Common flags: `-E` extended regex, `-v` invert (lines that do NOT match),
`-i` case-insensitive, `-o` print only the match, `-c` count, `-w` whole word.

```bash
grep -E "(alpha|beta)" file    # lines containing 'alpha' OR 'beta'
grep -E "alpha.*beta" file     # 'alpha' then later 'beta' (ordered AND)
grep -v "#" file               # lines that do NOT contain '#'
grep -E "^Start" file          # lines beginning with 'Start'
grep -E "end$" file            # lines ending with 'end'
grep -E "\bPre" file           # lines with a word starting 'Pre'
```

**Notes / gotchas**
- Without `-E`, the operators `| + ? { } ( )` are literal unless backslash-escaped — `-E` avoids the backslash soup.
- `.*` between two terms enforces order (first, then second); `|` cares about neither order nor requiring both.
- `-v "#"` drops every line containing `#` anywhere, including inline comments — not just full-line comments.
- Anchors and boundaries still match inside comment lines; combine with `grep -v "#"` or `^` when you want only active config.

### File permissions (rwx, chmod, chown)

Every file/directory has an **owner**, a **group**, and a permission set split
into three triads — owner, group, others — each with read (`r`), write (`w`) and
execute (`x`). `ls -l` shows them in the first column:

```
-rwxr-x---   →   type=-   owner=rwx   group=r-x   others=---
```

The leading character is the type: `-` file, `d` directory, `l` symlink.

What each bit means (differs for files vs directories):

| Bit | On a file | On a directory |
|-----|-----------|----------------|
| `r` | Read contents | List names (`ls`) |
| `w` | Modify contents | Create / delete / rename entries |
| `x` | Execute it | Traverse / enter (`cd`, reach items inside) |

You need `x` on a directory to enter it *at all* — with only `r` you can see names but can't reach the files (`Permission denied`). To delete a file you need `w` on its **directory**, not on the file itself.

**Octal notation** — each triad sums `r=4`, `w=2`, `x=1`:

| Octal | Bits | | Octal | Bits |
|-------|------|-|-------|------|
| 7 | rwx | | 3 | -wx |
| 6 | rw- | | 2 | -w- |
| 5 | r-x | | 1 | --x |
| 4 | r-- | | 0 | --- |

So `chmod 754` = `rwxr-xr--` (owner 7, group 5, others 4).

**Change permissions — `chmod`:**

```bash
chmod u+x file      # u=owner g=group o=others a=all ; + add, - remove
chmod a+r file      # add read for everyone
chmod o-w file      # remove write from others
chmod 640 file      # set exactly: rw-r-----
chmod -R 755 dir/   # recurse through a tree
```

**Change owner/group — `chown`:**

```bash
chown alice file           # owner only
chown alice:staff file     # owner and group
chown :staff file          # group only  (or: chgrp staff file)
chown -R alice:staff dir/  # recurse
```

**Special bits — SUID, SGID, sticky** (shown as `s`/`t` in place of `x`):

| Bit | Octal | Shows as | Effect |
|-----|-------|----------|--------|
| SUID | `4xxx` | `s` in owner's x | Run the file with the **owner's** privileges |
| SGID | `2xxx` | `s` in group's x | Run with the **group's** privileges; on a dir, new files inherit the dir's group |
| Sticky | `1xxx` | `t` in others' x | In a shared dir, only a file's owner (or root / dir owner) can delete or rename it |

```bash
chmod u+s file    # or 4755  -> -rwsr-xr-x
chmod g+s dir     # or 2775  -> drwxrwsr-x
chmod +t  dir     # or 1777  -> drwxrwxrwt   (e.g. /tmp)
```

**Notes / gotchas**
- Lowercase `s`/`t` means the underlying `x` is also set; **uppercase** `S`/`T` means the special bit is set but `x` is NOT — an uppercase `T` on a dir hides its contents from others.
- SUID-root binaries are a classic privilege-escalation vector: a program with a built-in shell escape, if SUID root, yields a root shell (see the public GTFOBins list). Audit them with `find / -perm -4000 -type f 2>/dev/null`.
- A new file's permissions are the base mode masked by your `umask` (`022` → `rwxr-xr-x` for dirs, `rw-r--r--` for files).

### Managing services with `systemctl`

`systemd` manages services as *units*; `systemctl` inspects and controls them.

```bash
systemctl list-units --type=service         # active service units
systemctl list-units --type=service --all   # incl. inactive / exited / failed
systemctl status <name>.service             # detailed state + recent logs
systemctl is-active <name>                   # or: is-enabled <name>
systemctl start|stop|restart|reload <name>  # control it now
systemctl enable|disable <name>             # start at boot (or not)
systemctl enable --now <name>               # enable + start in one step
systemctl --failed                          # list failed units
```

The columns in `list-units`: LOAD (unit file parsed), ACTIVE (running / exited /
failed), SUB (fine-grained sub-state), plus the unit's DESCRIPTION.

Service logs live in the journal:

```bash
journalctl -u <name>.service   # logs for one unit
journalctl -u <name> -f        # follow (tail) live
journalctl -b                  # since last boot
```

Inspect a unit's definition:

```bash
systemctl cat <name>.service    # print the full unit file
systemctl show -p Type <name>   # query a single property (e.g. Type=...)
```

Common `Type=` values (how systemd decides a service is "up"): `simple`
(default — the ExecStart process *is* the service), `forking` (daemonises then
the parent exits), `oneshot` (runs once and exits), `dbus` (ready once it claims
its D-Bus name), `notify` (signals readiness via `sd_notify`), `idle`.

**Notes / gotchas**
- A *oneshot* service can sit at ACTIVE=`active`, SUB=`exited` — it ran, did its job, and exited. That's normal, not a failure.
- `systemctl show <unit>` prints properties even for units that don't exist (empty `Type=`) — check `LoadState=not-found` to catch a missing/mistyped unit.

### Task scheduling — cron & systemd timers

Two ways to run tasks automatically on a schedule.

**cron** — each crontab line is five time fields then the command:

```
 ┌ minute (0-59)
 │ ┌ hour (0-23)
 │ │ ┌ day of month (1-31)
 │ │ │ ┌ month (1-12)
 │ │ │ │ ┌ day of week (0-7, both 0 and 7 = Sunday)
 │ │ │ │ │
 * * * * *  /path/to/command
```

```bash
crontab -e     # edit your user crontab
crontab -l     # list it
crontab -r     # remove it
```

| Schedule | Line |
|----------|------|
| Every 6 hours | `0 */6 * * * cmd` |
| 1st of month, midnight | `0 0 1 * * cmd` |
| Every Sunday, midnight | `0 0 * * 0 cmd` |
| Every 5 minutes | `*/5 * * * * cmd` |

System-wide jobs live in `/etc/crontab` and `/etc/cron.d/*` (these add a **user**
column before the command); drop-in scripts go in `/etc/cron.{hourly,daily,weekly,monthly}/`.

**systemd timers** — a `.timer` unit triggers a matching `.service` unit:

```ini
# /etc/systemd/system/mytask.timer
[Unit]
Description=Run mytask periodically
[Timer]
OnBootSec=3min             # once, this long after boot
OnUnitActiveSec=1h         # then repeatedly at this interval
# OnCalendar=*-*-* 02:00:00 # or an absolute schedule (daily 02:00)
[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/mytask.service
[Unit]
Description=My task
[Service]
Type=oneshot
ExecStart=/full/path/to/script.sh
```

```bash
sudo systemctl daemon-reload             # after editing unit files
sudo systemctl enable --now mytask.timer # enable the TIMER (not the service)
systemctl list-timers --all              # next/last run of every timer
```

**Notes / gotchas**
- The `.timer` and `.service` must share a base name (`mytask.timer` → `mytask.service`), or point elsewhere with `Unit=` in the `[Timer]` section.
- Run `daemon-reload` after any unit-file change, then `enable --now` the **timer**.
- Day-of-week `0` and `7` both mean Sunday.
- Test an `OnCalendar=` expression with `systemd-analyze calendar "<expr>"`.
- `/etc/cron.d` / `/etc/crontab` entries need an extra username column that personal (`crontab -e`) entries do not.

### Network services (SSH, NFS, web servers, VPN)

**SSH** — encrypted remote shell / file transfer (OpenSSH; server = `sshd`, port 22).

```bash
ssh user@host                 # remote shell
ssh user@host 'command'       # run a single command remotely
scp file user@host:/path/     # copy to remote  (reverse: scp user@host:/f .)
ssh-keygen -t ed25519         # generate a key pair
ssh-copy-id user@host         # install your pubkey for passwordless login
systemctl status ssh          # server state (config: /etc/ssh/sshd_config)
```

**NFS** — mount a remote directory as if it were local (server pkg `nfs-kernel-server`).

```bash
# server: add an export to /etc/exports, then reload:
#   /srv/share  <client>(rw,sync,no_root_squash)
sudo exportfs -ra
showmount -e <host>                 # list a server's exports
sudo mount <host>:/srv/share /mnt   # client: mount it
```

Export options: `rw` / `ro`, `sync` / `async`, and `root_squash` (map client
root → nobody, the default) vs `no_root_squash` (keep client root).

**Quick web server** — serve a directory for ad-hoc file transfer:

```bash
python3 -m http.server                    # serves ./ on port 8000
python3 -m http.server 80                 # pick the port
python3 -m http.server --directory /path  # serve a different dir
php -S 127.0.0.1:8080                      # PHP built-in server  (host:port)
http-server -p 8080                        # npm 'http-server' package  (-p = port)
# pull from the other side:  wget http://<host>:8000/file   |   curl -O http://<host>:8000/file
```

Apache serves `/var/www/html` by default (`apt install apache2`, config in
`/etc/apache2/`) — better for larger/persistent hosting; the Python one-liner is
the go-to for quick transfers.

**VPN** — join a remote network over an encrypted tunnel (OpenVPN):

```bash
sudo openvpn --config client.ovpn   # connect using a provided profile
ip a show tun0                       # the tunnel interface, once connected
```

**Notes / gotchas**
- FTP and Telnet send credentials in **cleartext** — prefer SSH / SCP / SFTP and HTTPS. Sniffing an FTP login hands over the password directly.
- `no_root_squash` lets a client's root write files as root on the server — a classic privilege-escalation path; keep `root_squash` unless you really need otherwise.
- `python3 -m http.server` is unauthenticated and exposes everything under the served directory — don't run it in a sensitive location or leave it up.
- Binding a service to a port below 1024 (e.g. `http.server 80`) requires root.

### Backups with `rsync`

`rsync` copies files locally or over SSH, transferring only the **changed parts**
of files — efficient for incremental backups and syncing.

```bash
rsync -av  src/ dest/                       # archive + verbose, local
rsync -avz src/ user@host:/backup/          # over SSH, compressed (-z)
rsync -avz -e ssh src/ user@host:/backup/   # force SSH explicitly (custom key/port)
rsync -av user@host:/backup/ dest/          # restore: pull from remote
```

| Flag | Effect |
|------|--------|
| `-a` | Archive: preserve perms, times, symlinks, ownership (implies `-rlptgoD`) |
| `-v` | Verbose progress |
| `-z` | Compress during transfer |
| `--delete` | Remove files in dest that no longer exist in src (mirror) |
| `--backup --backup-dir=DIR` | Keep changed/deleted files as an incremental copy |
| `-n` / `--dry-run` | Show what would happen without doing it |

**Trailing slash matters:** `rsync -a src/ dest/` copies the *contents* of `src`
into `dest`; `rsync -a src dest/` copies the `src` directory itself (creating `dest/src`).

Automate with cron + key-based SSH so it runs unattended:

```bash
ssh-keygen -t ed25519      # once: generate a key (empty passphrase for unattended use)
ssh-copy-id user@host      # install the pubkey on the backup server
# crontab line, e.g. hourly:
0 * * * * rsync -avz -e ssh /data/ user@host:/backup/
```

**Notes / gotchas**
- The trailing-slash rule is the #1 rsync surprise — always `--dry-run` first.
- `--delete` turns dest into a mirror; a wrong source path can wipe the backup. Dry-run it.
- Unattended cron rsync over SSH needs a passphrase-less key (or an ssh-agent), or it hangs waiting for input.
- For encrypted backups, layer a tool like duplicity (rsync + GnuPG) or back up onto a LUKS / eCryptfs volume.

### Disks, partitions & mounting

Inspect storage:

```bash
lsblk                              # tree of disks, partitions, mount points
lsblk -o NAME,TYPE,SIZE,MOUNTPOINT
sudo fdisk -l                      # partition tables (needs root)
df -h                              # free space per mounted filesystem
df -i                              # free INODES per filesystem
mount                              # list currently mounted filesystems
blkid                              # UUIDs + fs types of block devices
```

`lsblk` TYPE column: `disk` = whole device, `part` = partition, `loop`/`lvm` =
other. Count real partitions with `lsblk -o TYPE | grep -cw part`.

Mount / unmount:

```bash
sudo mount /dev/sdb1 /mnt/usb   # attach a device to a directory
sudo umount /mnt/usb            # detach (must not be in use)
lsof /mnt/usb                   # find what's keeping it busy
```

Persistent mounts live in `/etc/fstab`, one line per filesystem:

```
# <device/UUID>   <mount point>  <type>  <options>       <dump> <pass>
UUID=xxxx-xxxx    /              ext4    defaults          0      1
/dev/sdb1         /mnt/usb       ext4    rw,noauto,user    0      0
```

Prefer `UUID=` over `/dev/sdX` (device names can shift on reboot). `noauto`
skips mounting at boot; test fstab edits with `sudo mount -a`.

**Swap** — overflow RAM / hibernation space:

```bash
sudo mkswap /swapfile   # format a device or file as swap
sudo swapon /swapfile   # activate it   (swapon --show to list)
free -h                 # RAM + swap usage
```

**Notes / gotchas**
- A filesystem can exhaust its **inodes** (many tiny files) while `df -h` still shows free space — check `df -i`. (Each file consumes one inode; see the inode note above.)
- `umount` fails with "target is busy" when a process holds files open there — locate it with `lsof <mountpoint>` or `fuser -m <mountpoint>`.
- A broken `/etc/fstab` can block boot; validate with `sudo mount -a` before rebooting.
- Make swap permanent by adding the swap device/file to `/etc/fstab` as well.

### Containers (Docker & LXC)

Containers package an app with its dependencies and run it isolated while sharing
the host **kernel** (lighter than a VM). Isolation comes from kernel *namespaces*
(pid / net / mnt / …); resource limits come from *cgroups*.

**Docker** — build from a `Dockerfile`, run images as containers:

```bash
docker build -t myimg .              # build image from ./Dockerfile
docker run -d -p 8080:80 myimg       # run detached, map host:container port
docker run -it myimg bash            # interactive shell in a fresh container
docker ps           # running   (docker ps -a = all)
docker stop|start|restart <id|name>
docker rm <container>   /   docker rmi <image>
docker logs <container>              # container stdout/stderr
docker exec -it <container> bash     # shell into a running container
docker commit <container> myimg:tag  # snapshot a container to a new image
```

Minimal Dockerfile:

```dockerfile
FROM ubuntu:24.04
RUN apt-get update && apt-get install -y <pkgs> && rm -rf /var/lib/apt/lists/*
EXPOSE 80
CMD ["/usr/sbin/apache2ctl","-D","FOREGROUND"]
```

**LXC** — system-level containers that behave more like lightweight VMs:

```bash
sudo lxc-create -n name -t ubuntu   # create from a template
sudo lxc-start -n name              # start   (lxc-stop to stop)
sudo lxc-ls --fancy                 # list containers
sudo lxc-attach -n name             # get a shell inside
```

Limit resources via cgroup keys in the container config, e.g.
`lxc.cgroup.cpu.shares = 512` (relative CPU, default 1024) and
`lxc.cgroup.memory.limit_in_bytes = 512M`.

**Notes / gotchas**
- An image is a read-only template; a container is a running instance of one.
- Containers are **stateless** — changes vanish when the container is removed. Persist with `-v host:container` **volumes**, or bake changes into a new image (`docker commit` / rebuild).
- Port mapping is `-p HOST:CONTAINER`, and the process must actually listen on the container port inside.
- Sharing the host kernel means weaker isolation than a VM — a misconfig (privileged container, mounted docker socket, host bind-mounts) can enable container escape / host privesc.
- Docker builds on the same kernel features (namespaces + cgroups) as LXC, adding portable images and a simpler CLI.

### Network configuration & troubleshooting

Inspect / configure interfaces (`ip` is modern; `ifconfig` / `route` are legacy):

```bash
ip addr                       # interfaces + IPs           (legacy: ifconfig)
ip link set eth0 up|down      # bring an interface up/down (ifconfig eth0 up)
ip route                      # routing table             (route -n)
sudo ip addr add 192.168.1.2/24 dev eth0    # assign an IP
sudo ip route add default via 192.168.1.1   # set default gateway
```

DNS resolvers live in `/etc/resolv.conf` (`nameserver 8.8.8.8`), but that file is
usually auto-generated — set DNS persistently via NetworkManager / systemd-resolved
or the distro's network config (`/etc/network/interfaces`, netplan, …).

Troubleshooting toolkit:

| Tool | Use |
|------|-----|
| `ping <host>` | Reachability + round-trip time (ICMP) |
| `traceroute <host>` | Path + per-hop latency (`*` = no reply / ICMP blocked) |
| `dig <name>` / `nslookup <name>` | DNS resolution |
| `ss -tulpn` / `netstat -tulpn` | Local listening sockets & connections |
| `mtr <host>` | Live combined ping + traceroute |
| `curl -I <url>` | HTTP reachability / headers |

**Hardening / access control** — three complementary mechanisms:

| Mechanism | Scope |
|-----------|-------|
| SELinux | Kernel-integrated MAC; fine-grained per-process/file policies (powerful, complex) |
| AppArmor | MAC via per-application profiles (simpler, path-based) |
| TCP wrappers | Host-based service access by client IP (`/etc/hosts.allow`, `/etc/hosts.deny`) |

Access-control models: **DAC** (owner sets permissions — standard file perms),
**MAC** (kernel/policy enforced, e.g. SELinux/AppArmor), **RBAC** (permissions by role).

**Notes / gotchas**
- `ip` / `ifconfig` changes are runtime-only and lost on reboot — persist them in the distro's network config.
- `traceroute` hops showing `* * *` usually mean ICMP is filtered, not that the path is broken.
- "It's always DNS": when the IP is reachable but names aren't, check resolution (`dig`) early.
- `/etc/resolv.conf` edits are often overwritten by the network manager — change DNS at the manager level to make it persist.

### Remote graphical access — X11, VNC, RDP

Ways to reach a machine's GUI remotely.

**X11 forwarding** — run a remote GUI app rendered locally, tunneled over SSH:

```bash
ssh -X user@host /usr/bin/firefox   # needs 'X11Forwarding yes' in server sshd_config
```

X11 itself is unencrypted (raw ports TCP 6000-6010; display `:0` = 6000, `:1` =
6001, …) — always tunnel it through SSH. XDMCP (UDP 177) is likewise insecure.

**VNC** — share/control a full remote desktop (RFB protocol, TCP 5900 + display
number → `:1` = 5901, `:2` = 5902). Tools: TigerVNC, TightVNC, RealVNC, UltraVNC.

```bash
vncpasswd            # set the VNC password (~/.vnc/passwd)
vncserver            # start a session; prints display :1 and port 5901
vncserver -list      # show sessions, ports, PIDs
vncserver -kill :1   # stop a session
```

Tunnel VNC over SSH so it's encrypted, then point the viewer at localhost:

```bash
ssh -L 5901:127.0.0.1:5901 -N -f -l user host   # local -> remote port forward
xtigervncviewer localhost:5901
```

**RDP** — mainly Windows; connect from Linux with `xfreerdp /u:user /v:host` or `remmina`.

**Notes / gotchas**
- X11 renders on the *client* (saves load/traffic on the remote host); VNC/RDP render remotely and stream pixels.
- Open X11 ports (6000-6010) are a real exposure — an on-path attacker can screenshot windows (`xwd`) or read keystrokes. Keep it SSH-tunneled, never bare on the network.
- `ssh -L local:host:port` = local port forward; `-N` (run no command) and `-f` (background) are the usual tunnel flags.
- Port math: VNC display N → port 5900+N; X11 display `:N` → port 6000+N.

### Linux security hardening (basics)

A practical baseline for locking down a host:

- **Patch:** `apt update && apt full-upgrade` (kernel updates may need a reboot).
- **SSH:** in `/etc/ssh/sshd_config` set `PermitRootLogin no` and `PasswordAuthentication no` (use keys), then restart `ssh`.
- **Least privilege:** don't work as root; grant specific commands via `visudo` (`/etc/sudoers`) instead of blanket sudo.
- **Brute-force:** `fail2ban` bans IPs after repeated failed logins.
- **Firewall:** allow only the ports you need (`ufw` is the simple front end to iptables/nftables).
- **Accounts:** one per user, strong passwords, password aging (`chage`), lockout after failures.
- **Trim:** remove unused services/software and anything using unencrypted auth (telnet, FTP).
- **Audit:** `lynis` (hardening audit), `rkhunter` / `chkrootkit` (rootkits); hunt world-writable files and stray SUID/SGID with `find / -perm -4000 -o -perm -2000 2>/dev/null`.
- **Logging/time:** keep syslog/journald running and NTP enabled.

**Firewall (ufw) quickstart:**

```bash
sudo ufw default deny incoming
sudo ufw allow 22/tcp        # or: sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status numbered
```

**TCP wrappers** — per-service host access control, checked before the service
accepts the connection:

```
# /etc/hosts.allow        service : client(s)
sshd : 192.168.1.0/24
ftpd : 192.168.1.10
# /etc/hosts.deny
ALL  : ALL                # default-deny; put the allow-list in hosts.allow
```

**Notes / gotchas**
- TCP wrappers check `hosts.allow` first, then `hosts.deny`; first match wins, so a broad `ALL : ALL` deny only catches what wasn't already allowed.
- Wrappers gate *services* linked against libwrap, not *ports* — not a firewall replacement; use both.
- Security is a process, not a one-off — re-audit after changes. See also the SELinux / AppArmor / DAC-MAC-RBAC note above.

### Firewalls with `iptables`

Linux packet filtering runs on the kernel's **Netfilter** framework; `iptables`
(or the newer `nftables`, or front ends `ufw` / `firewalld`) configure it.
Structure: *tables* → *chains* → *rules* (match criteria → target).

| Table | Purpose | Key chains |
|-------|---------|-----------|
| filter | Allow/block traffic (default) | INPUT, OUTPUT, FORWARD |
| nat | Rewrite source/dest addresses (NAT) | PREROUTING, POSTROUTING |
| mangle | Modify packet headers | PRE/POSTROUTING, INPUT, OUTPUT, FORWARD |
| raw | Special/early processing | PREROUTING, OUTPUT |

Common targets: `ACCEPT`, `DROP` (silent), `REJECT` (with error), `LOG`,
`SNAT` / `MASQUERADE`, `DNAT`, `REDIRECT`.

**Managing rules:**

```bash
sudo iptables -L -n -v --line-numbers                 # list (numeric, counters, line #s)
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT    # append a rule
sudo iptables -I INPUT 1 -s 1.2.3.4 -j DROP           # insert at position 1
sudo iptables -D INPUT 3                               # delete rule #3 in INPUT
sudo iptables -P INPUT DROP                            # default policy for a chain
sudo iptables -N mychain                               # create a user-defined chain
sudo iptables -F                                       # flush all rules
```

Common matches: `-p tcp|udp|icmp`, `--dport` / `--sport`, `-s` / `-d` (address),
`-i` / `-o` (in/out interface), `-m state --state NEW,ESTABLISHED,RELATED`,
`-m multiport --dports 80,443`, `-m iprange`, `-m mac`.

Stateful allow-list skeleton:

```bash
sudo iptables -P INPUT DROP
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

**Notes / gotchas**
- Rule order matters — **first match wins**. Put specific rules before broad ones; use the chain policy (`-P ... DROP`) as the catch-all.
- Allow loopback (`-i lo`) and `ESTABLISHED,RELATED` early, or you'll cut your own return traffic — and lock yourself out of SSH.
- `iptables` rules are **not persistent** — save with `iptables-save` / the `iptables-persistent` package, or they vanish on reboot.
- `DROP` is silent (the client hangs); `REJECT` returns an error immediately — DROP is stealthier, REJECT is easier to debug.
- On a remote box, stage a timed rollback (e.g. an `at`/cron job that flushes rules) before applying, so a bad rule doesn't strand you.

### System logs

Where things get logged and how to read them. Modern systemd distros centralise
logs in the **journal** (`journalctl`); many services also write plain-text files
under `/var/log`.

| Path | Contents |
|------|----------|
| `/var/log/syslog` (Debian) / `messages` (RHEL) | General system events |
| `/var/log/kern.log` | Kernel: drivers, hardware, syscalls |
| `/var/log/auth.log` (Debian) / `secure` (RHEL) | Auth: logins, sudo, sshd |
| `/var/log/apache2/`, `/var/log/nginx/` | Web server access/error logs |
| `/var/log/mysql/`, `/var/log/postgresql/` | Database logs |
| `/var/log/fail2ban.log`, `/var/log/ufw.log` | Security tooling |
| `/var/log/journal/` | Binary systemd journal |

Reading logs:

```bash
tail -f /var/log/syslog                       # follow a text log live
grep -i "failed password" /var/log/auth.log   # search a log
less /var/log/syslog                          # page through
journalctl -xe                                # recent journal + explanations
journalctl -u ssh --since "1 hour ago"        # one unit, time-filtered
journalctl -k                                 # kernel messages (= dmesg)
journalctl -p err -b                          # only errors, this boot
```

**Notes / gotchas**
- On systemd distros the journal is the source of truth; `rsyslog` may or may not also mirror it to `/var/log/*` text files — check both.
- `logrotate` (`/etc/logrotate.conf`, `/etc/logrotate.d/`) rotates & compresses logs so they don't fill the disk; older ones become `*.1`, `*.2.gz`, …
- The journal is binary — read it with `journalctl`, not `cat`. Make it persistent by creating `/var/log/journal/` (otherwise it's memory-only and lost on reboot).
- `auth.log` / `secure` is the first place to look for logins and privilege (sudo) use.

### Solaris vs Linux — quick command map

Solaris is a proprietary Unix (Sun/Oracle). Common tasks differ from Linux:

| Task | Linux | Solaris |
|------|-------|---------|
| System info | `uname -a` | `showrev -a` |
| Install package | `apt-get install <pkg>` | `pkgadd -d <pkg>` |
| Package system | APT / dpkg | IPS (`pkg`) / SVR4 (`pkgadd`) |
| Service management | `systemctl` | SMF (`svcs`, `svcadm`) |
| Files open by a process | `lsof -c <name>` | `pfiles $(pgrep <name>)` |
| Trace system calls | `strace -p <pid>` | `truss <cmd>` |
| Share via NFS | `/etc/exports` + `exportfs` | `share -F nfs -o rw <dir>`, `/etc/dfs/dfstab` |

**Notes / gotchas**
- `find / -perm -4000` (leading `-`) means "has at least these bits" (SUID set); `find / -perm 4000` means "exactly this mode". Same rule on GNU find — the `-` is the key, not a Solaris-only quirk.
- Solaris leans on **RBAC** for privilege (granular authorizations) rather than `sudo` (though sudo exists since Solaris 11).
- `truss` can also trace signals and child-process syscalls; `strace` needs `-f` to follow children.
- Default shell and `ps` / `ls` flags differ from GNU — check `man` on the actual box rather than assuming GNU behaviour.

### Terminal shortcuts (readline)

Bash line editing uses readline keybindings — worth memorising to avoid the mouse.

| Shortcut | Action |
|----------|--------|
| `Tab` | Auto-complete command / path / option |
| `Ctrl`+`A` / `Ctrl`+`E` | Jump to start / end of line |
| `Alt`+`B` / `Alt`+`F` | Move back / forward one word |
| `Ctrl`+`U` / `Ctrl`+`K` | Delete to start / end of line |
| `Ctrl`+`W` | Delete the word before the cursor |
| `Ctrl`+`Y` | Paste (yank) the last deleted text |
| `Ctrl`+`R` | Reverse-search command history |
| `↑` / `↓` | Previous / next command |
| `Ctrl`+`L` | Clear the screen (= `clear`) |
| `Ctrl`+`C` | Kill the current process (SIGINT) |
| `Ctrl`+`Z` | Suspend to background (SIGTSTP; resume with `fg`) |
| `Ctrl`+`D` | Send EOF / close stdin (exits an empty shell) |

**Notes / gotchas**
- `Ctrl`+`Z` only *suspends* — the job is paused, not gone; `jobs` lists it, `fg` / `bg` resume it in the fore/background.
- `Ctrl`+`S` freezes terminal output (XOFF), `Ctrl`+`Q` resumes it (XON) — worth knowing if the terminal seems to hang.
- The delete shortcuts feed a "kill ring"; `Ctrl`+`Y` pastes the most recent kill.
- `enable` only affects boot; `start` only affects now — use `enable --now` for both at once.
- Find a unit by its description text: `systemctl list-units --type=service --all | grep -i "<text>"`.
