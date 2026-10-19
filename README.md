# 📂 dlsort

**Your Downloads folder, sorted automatically.**

dlsort quietly watches your Downloads folder and, the moment a download finishes, moves the file where it belongs: pictures to Pictures, documents to Documents, music to Music, and so on. It's one small Bash script, with no daemon to configure and nothing to compile. You control it with simple commands: `start`, `stop`, `status`.

```text
~/Downloads/holiday.jpg      →  ~/Pictures/holiday.jpg
~/Downloads/invoice.pdf      →  ~/Documents/invoice.pdf
~/Downloads/song.mp3         →  ~/Music/song.mp3
~/Downloads/backup.zip       →  ~/Documents/Archives/backup.zip
```

## ⚡ Quick start

```bash
# 1. Install the one dependency (Debian/Ubuntu/Mint shown, other distros below)
sudo apt install inotify-tools

# 2. Get dlsort and install it
git clone https://github.com/nJetto/dlsort.git
cd dlsort
install -Dm755 dlsort ~/.local/bin/dlsort

# 3. Start sorting!
dlsort start
```

That's it. Download something and watch it land in the right folder. 🎉

A few handy follow-ups:

```bash
dlsort once      # tidy up files that are already sitting in Downloads
dlsort enable    # start automatically every time you log in
dlsort status    # see what it's doing
```

> **`dlsort: command not found`?** Make sure `~/.local/bin` is in your `PATH`. Most distros add it automatically after you log out and back in. Or run it directly with `./dlsort start`.

## 🎛️ Commands

| Command | What it does |
| --- | --- |
| `dlsort start` | Start watching Downloads in the background |
| `dlsort stop` | Stop watching |
| `dlsort restart` | Restart the watcher |
| `dlsort status` | Show whether it's running, autostart state, and the latest moves |
| `dlsort once` | Sort what's already in Downloads, then exit |
| `dlsort log` | Follow the log live (Ctrl+C to quit) |
| `dlsort enable` | Start automatically at login |
| `dlsort disable` | Remove autostart |
| `dlsort version` | Print the version |
| `dlsort help` | Show built-in help, including the folders it detected on your system |

## 🗂️ What goes where

| Type | Extensions | Destination |
| --- | --- | --- |
| Images | jpg, jpeg, png, gif, webp, bmp, svg, heic, tif, tiff, avif, ico | Pictures |
| Videos | mp4, mkv, avi, mov, webm, flv, wmv, m4v | Videos |
| Music | mp3, flac, wav, ogg, m4a, opus, aac, wma | Music |
| Documents | pdf, doc, docx, odt, xls, xlsx, ods, ppt, pptx, odp, txt, rtf, epub, fb2, djvu, csv, md | Documents |
| Archives | zip, rar, 7z, tar, gz, xz, bz2, tgz, zst | Documents/Archives |
| Everything else | `.deb`, `.iso`, `.AppImage`, … | stays in Downloads |

Folder locations come from your system's user directories (`xdg-user-dir`), so localized folder names such as `Загрузки` or `Bilder` work out of the box. If your system doesn't define them, dlsort falls back to `~/Downloads`, `~/Documents`, `~/Pictures`, `~/Videos` and `~/Music`.

## 🖥️ Supported systems

| System | Status |
| --- | --- |
| Debian, Ubuntu, Linux Mint, Pop!_OS and derivatives | ✅ Supported |
| Fedora, RHEL-family | ✅ Supported |
| Arch, Manjaro, EndeavourOS | ✅ Supported |
| openSUSE | ✅ Supported |
| Other Linux distributions | ✅ Should work if the requirements below are met |
| Windows via WSL2 | ⚠️ Works on the Linux side. Watching Windows folders under `/mnt/c` is unreliable because inotify doesn't see changes there |
| macOS, BSD | ❌ Not supported (no inotify, and the script relies on Linux tools) |

**Requirements**

- Linux with a Bash **4.0+** shell (every current distro ships this)
- `inotify-tools` (provides `inotifywait`)
- `setsid` (part of `util-linux`, preinstalled almost everywhere)
- Optional: `xdg-user-dirs` for localized folder names

**Install the dependency**

| Distro | Command |
| --- | --- |
| Debian / Ubuntu / Mint | `sudo apt install inotify-tools` |
| Fedora | `sudo dnf install inotify-tools` |
| Arch / Manjaro | `sudo pacman -S inotify-tools` |
| openSUSE | `sudo zypper install inotify-tools` |

**Autostart** (`dlsort enable`) uses the standard XDG autostart mechanism, so it works on KDE Plasma, GNOME, XFCE, Cinnamon, MATE, LXQt and anything else that honors `~/.config/autostart`. If you use a bare window manager (i3, Sway, Hyprland, …), skip `enable` and add `dlsort start` to your WM's startup config instead.

## 🛠️ Make it yours

Open the script (`~/.local/bin/dlsort`) and find the `dest_for()` function near the top. It's a plain `case` block:

```bash
mp3|flac|wav|ogg|m4a|opus|aac|wma)
    echo "$DIR_MUSIC" ;;
```

- **Add a type:** put the extension in the list, e.g. `svg|psd|xcf`.
- **Use a different folder:** replace the destination, e.g. `echo "$HOME/Work/Invoices"`.
- **Don't touch a type:** delete its line (for example, the archives rule).

Then run `dlsort restart`. Destination folders are created automatically if they don't exist.

## 🔒 Safe by design

- **Never overwrites.** If `photo.png` already exists at the destination, the new file becomes `photo (1).png`.
- **Never deletes.** Files are only moved.
- **Waits for downloads to finish.** Partial files (`.part`, `.crdownload`, `.tmp`, …) are ignored until the browser renames them to the final name.
- **Top level only.** Subfolders in Downloads and hidden files are left alone.
- **Offline.** No network access, no telemetry, no dependencies beyond the ones above.

## 🔍 How it works

`dlsort start` launches a background loop built on `inotifywait`. It reacts to two events: a file finishing a write, and a file being renamed into Downloads. Each new file is matched against the rules, moved, and logged. `dlsort stop` ends the whole process group, so nothing is left running.

The log lives at `~/.local/state/dlsort/dlsort.log`.

## 🩹 Troubleshooting

| Problem | Fix |
| --- | --- |
| `inotifywait not found` | Install `inotify-tools` (see above) |
| `dlsort: command not found` | Add `~/.local/bin` to your `PATH`, or run the script by its full path |
| Nothing gets moved | Run `dlsort status`, then `dlsort log` and download a small file to watch what happens |
| Files land in the wrong place | Check the rules in `dest_for()`, then `dlsort restart` |
| Your browser saves elsewhere | dlsort watches the system Downloads directory. Point your browser at it or change `SRC` in the script |
| `Already running` | That's fine. Use `dlsort restart` if you edited the script |

## 🗑️ Uninstall

```bash
dlsort stop
dlsort disable
rm ~/.local/bin/dlsort
rm -r ~/.local/state/dlsort    # optional: removes the log
```

## 🤝 Contributing

Bug reports, ideas and pull requests are welcome. If dlsort misbehaves on your distro, please open an issue and include the output of `dlsort version` and `dlsort status`.
