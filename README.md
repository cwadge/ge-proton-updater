# GE-Proton Updater

A Bash script to automatically download, verify, and install the latest [GE-Proton](https://github.com/GloriousEggroll/proton-ge-custom) ([GloriousEggroll](https://x.com/GloriousEggroll)'s excellent Proton fork) for Steam on Linux desktops / laptops. Keeps your gaming compatibility tools up-to-date with minimal hassle.

- **Standalone**: Runs anywhere to update GE-Proton.
- **Hook-Friendly**: Perfect as a post-update script for [apt-up](https://github.com/cwadge/apt-up) or [pac-up](https://github.com/cwadge/pac-up).
- **Efficient**: Nothing happens unless it needs to.

## Features
- Fetches the latest GE-Proton from GitHub.
- Verifies checksums for security and consistency.
- Installs to `~/.steam/steam/compatibilitytools.d` (or a custom path).
- Cleans up old versions (keeps latest + 1 previous).
- Handles user ownership (no root-owned files in your home dir).
- Skips root installs as accidental unless explicitly allowed (`--force-root`).
- Debug mode for troubleshooting (`--debug`)

## Prerequisites
- `Bash`, `curl`, `tar`, `sha512sum` (standard on most distros).
- `Steam` installed (native or Flatpak).

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

- **Force Root Install:** `ge-proton-updater --force-root` (if you're running Steam as root for some wild and unforseen reason).

- **Debug Mode:** Something going wrong? Run `ge-proton-updater --debug` for full verbiage.

## Steam Setup
After a new GE-Proton install, Steam needs to be told to use it:

1. Restart Steam.

2. Go to `Steam` > `Settings` > `Compatibility`.

3. Check "Enable Steam Play for all other titles".

4. Set "Run other titles with" to the new `GE-Proton` version (e.g. `GE-Proton9-27`).

## Multi-User, Multi-Steam Systems
For systems with multiple Steam users, adapt the script as follows:

### Admin And Gamer Aren't The Same User:
Set `GE_PROTON_TARGET` upstream for the gamer’s Steam path, e.g.:
1. In `/etc/apt-up.conf` or `/etc/pac-up.conf`:
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

2. Run parent script, e.g. `sudo apt-up` or `sudo pac-up`. Each hook updates its respective user’s GE-Proton individually.

    - Be sure you don’t set `GE_PROTON_TARGET` in `apt-up.conf`/`pac-up.conf` in this case, as it’d override the script-specific targets.

## License
MIT License ([LICENSE](https://opensource.org/license/MIT)) - feel free to use, modify, and share!

---

_A script for keeping GE-Proton up to date by Chris Wadge_
