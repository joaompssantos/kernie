# Kernie

Kernie - Kernel Genie, a Gentoo kernel manager.

A `dialog` front end (or plain command line) for the manual kernel routine on Gentoo with `sys-kernel/gentoo-sources` or any other `*-sources` package (kernie asks Portage which package owns each tree): select the sources, carry the configuration over, answer the new options, build, install, and remove old kernels. It works with or without an initramfs.

## Features

- **Upgrade** to new sources: saves the running and previous configurations, starts the new tree from the one you choose, shows how many options are new, then runs `make oldconfig` (asks about each) or `make olddefconfig`, with an optional `menuconfig` review.
- **Build and install** in one go: `make` with Portage's `MAKEOPTS` and niceness, `make modules_install`, `emerge @module-rebuild`, `make install`, then the initramfs and GRUB as needed. Stops at the first failed step.
- **Initramfs or not:** follows `sys-kernel/installkernel` (dracut or ugrd flag), runs its own command, or does without one (see below).
- **Checks** after every installation that the kernel, the initramfs and the GRUB entry are in place.
- **Delete** kernels completely: `/boot` files (initramfs and `.old` copies included), modules, sources and their world entry. Or remove just a `.old` previous build, or a source tree with no installed kernel (an unbuilt sources slot, or build files left in `/usr/src` after `emerge --depclean`). The running kernel and the selected sources are never offered.
- **Keeps sources** of the kernels you build in the world file (with their slot), so `emerge --depclean` doesn't remove them.
- **Logs** every step and its output to `/var/log/kernie.log`, and offers to show it when a step fails.

## Requirements

- `app-admin/eselect`, `sys-apps/portage` (`portageq`), `dev-build/make`
- `dev-util/dialog`, optional: without it, or with `-n`, kernie uses plain text prompts
- `sys-kernel/installkernel`, recommended: with `USE="dracut grub"`, `make install` also builds the initramfs and updates GRUB
- `app-admin/eclean-kernel`, optional: only for the `autoclean` entry
- GRUB as the boot loader, if kernie should update its menu

## Installation

Through the ebuild (`sys-kernel/kernie`), or by hand:

```bash
install -m 0755 kernie /usr/local/sbin/kernie
install -m 0644 files/kernie.conf /etc/kernie.conf
```

## Usage

```
kernie [-n|--no-dialog] [command]
```

Without a command, kernie shows its menu. Run it as root.

| Command | What it does |
| --- | --- |
| `status` | Running kernel, sources, initramfs mode, `installkernel` flags, who updates GRUB |
| `select` | Point `/usr/src/linux` at other sources (`eselect kernel set`) |
| `upgrade` | New sources: configuration, new options, build, install |
| `rebuild` | Change the configuration (`menuconfig`), then build and install |
| `build` | Build and install with the current configuration |
| `initramfs` | Rebuild the initramfs of an installed kernel, then update GRUB |
| `grub` | Run `grub-mkconfig` |
| `clean` | `make clean` in the sources (keeps `.config`) |
| `delete` | Remove kernels completely, `.old` builds, or source trees without an installed kernel |
| `autoclean` | Keep the newest kernels with `eclean-kernel` |
| `log` | View the end of the log |

## Configuration

`/etc/kernie.conf`, documented in [`files/kernie.conf`](files/kernie.conf); every setting is optional. The one to look at first is `INITRAMFS`:

| Value | Who builds the initramfs |
| --- | --- |
| `auto` (default) | `installkernel`, if it has the `dracut` or `ugrd` flag; otherwise none |
| `installkernel` | `installkernel`, during `make install` |
| `kernie` | kernie, with `INITRAMFS_CMD` (default `dracut --force --kver %KVER%`), then `grub-mkconfig` |
| `no` | Nobody: the disk and filesystem drivers must be built in (`=y`) |

Settings from kernie 1 still work: `GEN_INITRAMFS=1` means `INITRAMFS=kernie`, and `GEN_INITRAMFS_CMD` is used as `INITRAMFS_CMD` if that is unset.

## To do

- Test on more systems: kernie 2 has so far been checked with `shellcheck` and against stand-in tools.
- Boot loaders other than GRUB (systemd-boot, rEFInd, EFI stub or UKIs). `installkernel` already supports them; kernie would need to stop assuming `grub-mkconfig` (`UPDATE_GRUB=no` works around it for now).
- A dry-run mode that shows the commands without running them.

## License

GPL-3.0-or-later, see [LICENSE](LICENSE).
