# rlconf-sync

Keep rEFInd's `/boot/refind_linux.conf` in sync with Fedora's
`/etc/kernel/cmdline` using a small Bash script and a systemd path unit.

The generator writes three boot entries:

* Standard options: the full command line.
* Single-user mode: the full command line with `single` appended.
* Minimal options: the command line with the first ` rhgb` and everything
  following it removed. Put options you want to keep before `rhgb`.

The file is replaced atomically, with mode `0644`.

## Installation

Requires Bash, GNU coreutils, systemd, an existing `/etc/kernel/cmdline`,
and a mounted, writable `/boot` used by rEFInd. Run from this directory:

```sh
sudo install -m 0755 update-rlconf /usr/local/bin/update-rlconf
sudo install -m 0644 rlconf.service rlconf.path /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now rlconf.path
sudo systemctl start rlconf.service
```

The final command generates the initial configuration, replacing any existing
`/boot/refind_linux.conf`. Save a copy first if it contains custom entries.
The path unit then regenerates it whenever `/etc/kernel/cmdline` changes.

## Usage

Regenerate manually:

```sh
sudo systemctl start rlconf.service
```

Inspect the watcher and generator logs:

```sh
systemctl status rlconf.path
journalctl -u rlconf.service
```

This watches only `/etc/kernel/cmdline`; edits made only to BLS entries or
`/etc/default/grub` do not trigger a sync.

## Uninstallation

```sh
sudo systemctl disable --now rlconf.path
sudo systemctl stop rlconf.service
sudo rm /etc/systemd/system/rlconf.path /etc/systemd/system/rlconf.service
sudo rm /usr/local/bin/update-rlconf
sudo systemctl daemon-reload
```

The generated `/boot/refind_linux.conf` is left in place.

