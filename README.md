# GE-Proton Updater

A Bash script to automatically download, verify, and install the latest [GE-Proton](https://github.com/GloriousEggroll/proton-ge-custom) ([GloriousEggroll](https://x.com/GloriousEggroll)'s excellent Proton fork) for Steam on Linux desktops / laptops. Keeps your gaming compatibility tools up-to-date with minimal hassle.

- **Standalone**: Runs anywhere to update GE-Proton.
- **Hook-Friendly**: Perfect as a post-update script for [apt-up](https://github.com/cwadge/apt-up) or [pac-up](https://github.com/cwadge/pac-up).
- **Efficient**: Nothing happens unless it needs to.
- **No Restart Required**: After initial setup, GE-Proton updates are picked up by Steam automatically — no restart needed.

## Features
- Fetches the latest GE-Proton from GitHub.
- Verifies checksums for security and consistency.
- Installs to a **static** `GE-Proton-Latest` directory in `~/.steam/steam/compatibilitytools.d` (or a custom path), with a customized `compatibilitytool.vdf` so Steam recognizes it as **"Proton GE Latest"**.
- Automatically rotates the previous version to `GE-Proton-Previous` (**"Proton GE Previous"**) as a fallback in case of critical bugs.
- After initial setup, **no Steam restart is required** for updates — the static internal name means Steam uses the new build immediately.
- Tracks installed versions via a state file (`.ge-proton-version`) to avoid redundant downloads.
- Cleans up legacy versioned directories (e.g. `GE-Proton10-27`) from older script versions.
- Handles user ownership (no root-owned files in your home dir).
- Skips root installs as accidental unless explicitly allowed (`--force-root`).
- Debug mode for troubleshooting (`--debug`).

## How It Works
Instead of extracting each GE-Proton release to its own versioned directory (e.g. `GE-Proton10-30`), this script uses **static directory names** with rewritten `compatibilitytool.vdf` files:

| Directory | Internal Name | Display Name (in Steam) |
|---|---|---|
| `GE-Proton-Latest/` | `GE-Proton-Latest` | `Proton GE Latest` |
| `GE-Proton-Previous/` | `GE-Proton-Previous` | `Proton GE Previous` |

Since Steam's `CompatToolMapping` in `config.vdf` references tools by their **internal name**, keeping the internal name static means you only need to select `Proton GE Latest` once. After that, every update transparently replaces the contents of that directory and Steam uses the new build at the next game launch.

The actual GE-Proton version installed under each slot is tracked internally via a `.ge-proton-version` state file and reported in the script's output.

## Prerequisites
- `Bash`, `curl`, `tar`, `sha512sum` (standard on most distros).
- `Steam` installed natively (`.deb`, package manager, etc.).

> **Flatpak Steam users**: This script is not compatible with Flatpak Steam. Upstream GE-Proton tarballs include pressure-vessel, which cannot nest inside Flatpak's sandbox. Install GE-Proton via Flatpak instead:
> ```bash
> flatpak install com.valvesoftware.Steam.CompatibilityTool.Proton-GE
> ```

## Example Run
Here it's found a new version, rotated the previous Latest to Previous, downloaded and checksummed the new release, extracted it, and rewrote the VDF files:
```
Checking for GE-Proton updates...
Using overridden Steam directory: /home/chris/.steam/steam/compatibilitytools.d
Latest version: GE-Proton10-30
Rotating GE-Proton-Latest (GE-Proton10-29) → GE-Proton-Previous...
Previous version: GE-Proton10-29
Downloading GE-Proton10-30.tar.gz...
/tmp/ge-proton-update.BJfprL/GE-Proton10-30.tar.gz          100% 478.83M 90.1MB/s in 5.2s
Verifying checksum...
Checksum verification passed
Extracting GE-Proton10-30 to /home/chris/.steam/steam/compatibilitytools.d/GE-Proton-Latest as user chris...
GE-Proton10-30 installed as 'Proton GE Latest'!
==> Updated in-place. No Steam restart required.
Done!
```

## Installation
### Standalone
```bash
curl -sL https://raw.githubusercontent.com/cwadge/ge-proton-updater/main/ge-proton-updater -o /usr/local/bin/ge-proton-updater
chmod +x /usr/local/bin/ge-proton-updater
ge-proton-updater
```

### As an `apt-up`/`pac-up` Hook
1. Copy to the `apt-up` post-hook directory:
```bash
sudo curl -sL https://raw.githubusercontent.com/cwadge/ge-proton-updater/main/ge-proton-updater -o /etc/apt-up.d/post.d/50-ge-proton
sudo chmod +x /etc/apt-up.d/post.d/50-ge-proton
```

Or for `pac-up`:
```bash
sudo curl -sL https://raw.githubusercontent.com/cwadge/ge-proton-updater/main/ge-proton-updater -o /etc/pac-up.d/post.d/50-ge-proton
sudo chmod +x /etc/pac-up.d/post.d/50-ge-proton
```
2. (Optional) Set a custom target in `/etc/apt-up.conf` or `/etc/pac-up.conf`:
```bash
EXPORT_VARS="GE_PROTON_TARGET=/home/youruser/.steam/steam/compatibilitytools.d"
```

3. Run `sudo apt-up` or `sudo pac-up`.

## Usage
- **Standalone:** `ge-proton-updater` (updates for the current user).

- **Root with Custom Target:** `GE_PROTON_TARGET=/path/to/dir sudo ge-proton-updater`.

- **Force Root Install:** `ge-proton-updater --force-root` (if you're running Steam as root for some wild and unforeseen reason).

- **Debug Mode:** Something going wrong? Run `ge-proton-updater --debug` for full verbiage.

## Steam Setup
After the **first** GE-Proton install, Steam needs to be told to use it:

1. Restart Steam.

2. Go to `Steam` > `Settings` > `Compatibility`.

3. Set "Default compatibility tool:" to `Proton GE Latest`.

**That's it — you only need to do this once.** Future updates will be picked up automatically without restarting Steam.

If a new build causes issues, you can switch to `Proton GE Previous` as a fallback until the next release.

## Migrating from Older Versions
If you were previously using versioned directory names (e.g. `GE-Proton10-27`, `GE-Proton10-28`), the script will:

1. Install the latest version to `GE-Proton-Latest/` with the new static naming.
2. Automatically clean up any legacy versioned directories.

After migrating, you'll need to restart Steam once and select `Proton GE Latest` as your compatibility tool. From that point forward, updates are seamless.

## Multi-User, Multi-Steam Systems
For systems with multiple Steam users, adapt the script as follows:

### Admin And Gamer Aren't The Same User:
1. Set `GE_PROTON_TARGET` upstream for the gamer's Steam path,
e.g. edit `/etc/apt-up.conf` or `/etc/pac-up.conf`:
```bash
EXPORT_VARS="IGNORE_CC_MISMATCH=1 GE_PROTON_TARGET=/home/gamer/.steam/steam/compatibilitytools.d"
```

2. Run `sudo apt-up` or `sudo pac-up`.

### Standalone, Multiple Gamers:
Run the script once per user with `GE_PROTON_TARGET` as a CLI variable:
```bash
GE_PROTON_TARGET=/home/gamer1/.steam/steam/compatibilitytools.d ge-proton-updater
GE_PROTON_TARGET=/home/gamer2/.steam/steam/compatibilitytools.d ge-proton-updater
...
```

### Called By A Hook In An Update Script, Multiple Gamers:
Use multiple copies of the script with hard-coded targets:

1. Copy and edit the script for each user:
```bash
sudo cp ge-proton-updater /etc/apt-up.d/post.d/ge-proton-updater-gamer1
sudo chmod +x /etc/apt-up.d/post.d/ge-proton-updater-gamer1
# Edit ge-proton-updater-gamer1, uncomment and set: GE_PROTON_TARGET="/home/gamer1/.steam/steam/compatibilitytools.d"
sudo cp ge-proton-updater /etc/apt-up.d/post.d/ge-proton-updater-gamer2
sudo chmod +x /etc/apt-up.d/post.d/ge-proton-updater-gamer2
# Edit ge-proton-updater-gamer2, uncomment and set: GE_PROTON_TARGET="/home/gamer2/.steam/steam/compatibilitytools.d"
...
```

2. Run parent script, e.g. `sudo apt-up` or `sudo pac-up`. Each hook updates its respective user's GE-Proton individually.

    - Be sure you don't set `GE_PROTON_TARGET` in `apt-up.conf`/`pac-up.conf` in this case, as it'd override the script-specific targets.

## License
MIT License ([LICENSE](https://opensource.org/license/MIT)) - feel free to use, modify, and share!

---

_A script for keeping GE-Proton up to date by Chris Wadge_
