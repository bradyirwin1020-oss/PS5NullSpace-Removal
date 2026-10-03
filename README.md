<div align="center">

# NULLSPACE

### Game Remover for jailbroken PlayStation 5 consoles

**Browse your stored games. See their artwork. Remove what you no longer need.**

![Version](https://img.shields.io/badge/version-v0.5-8b5cf6?style=flat-square)
![Platform](https://img.shields.io/badge/platform-PS5-2563eb?style=flat-square)
![Firmware](https://img.shields.io/badge/tested%20setup-13.42-334155?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-22c55e?style=flat-square)

*A locally hosted, PS5-friendly game source manager with a dedicated Media-section shortcut.*

</div>

---

> [!IMPORTANT]
> NULLSPACE is a **homebrew tool for an already-jailbroken PS5**. It does not jailbreak your console, make the jailbreak permanent, or remove every kind of installed PS5 application. **Deleting a game source is permanent.** Read [How deletion works](#how-deletion-works) before using it.

## Overview

Deleting a mounted game from the PS5 home screen does not necessarily delete the original game image. If a loader such as ShadowMountPlus can still find that image, the game may show up again.

**NULLSPACE Game Remover** helps you locate recognized game sources, identify them using available title metadata and artwork, and remove selected sources through an interface designed for the PS5. When the tool can **verify an exact link** between a source and a PS5 library entry, it can also request that the matching library entry be uninstalled.

NULLSPACE runs an HTTP interface **locally on the console** at `http://127.0.0.1:8877/` and registers a **NULLSPACE Game Remover** launcher in the PS5's **Media** section.

## Features

- **Game library interface** — dark, TV-friendly design with large game cards and controller-friendly buttons.
- **Actual game titles where available** — reads accessible `param.json` metadata; marks filename-derived titles as unverified.
- **Local game artwork** — displays available `icon0.png` covers, with a clean placeholder when no compatible image is found.
- **Search and filters** — find games by title, title ID, filename or path, and filter verified library links from source-only entries.
- **Source details** — review the selected game's title ID, file path and size before removal.
- **Simple confirmation** — choose **Manage**, review the game and press **Delete game** or **Delete source**. No typing required.
- **Optional library cleanup** — requests a PS5 uninstall only when the installed title has an exact, verified ShadowMount source link.
- **Offline local interface** — no external artwork service or internet connection is needed for the UI.
- **Media shortcut** — installs a launcher with its own icon (title ID `NSRM01342`).
- **Diagnostics** — startup and installation messages are written to `/data/nullspace_remover/startup.log`.

<!-- Add a real screenshot to docs/screenshot.png, then uncomment the next line. -->
<!-- ![NULLSPACE game library](docs/screenshot.png) -->

## Requirements

| Requirement | Details |
| --- | --- |
| Console | PlayStation 5 with a working jailbreak |
| Known tested setup | Firmware **13.42** with Relapse and a compatible ELF loader |
| Payload | `nullspace-remover-ps5.elf` (compiled for PS5) |
| Launcher | Compatible PS5 ELF loader or Payload Manager |
| Browser | PS5 WebKit, opened through the NULLSPACE Media shortcut or the local URL |
| Build from source | WSL/Linux, Python 3, `make`, and the [PS5 Payload SDK](https://github.com/ps5-payload-dev/sdk) |

**Compatibility note:** the development and user-tested setup is PS5 13.42. Other firmware versions and combinations of jailbreaks, HENs and game loaders have not been verified. The SDK and source code do not guarantee wider compatibility.

## Installation

### Option A — Use a compiled ELF

If a compiled ELF is provided in this repository's [Releases](../../releases):

1. Start your PS5 and run your usual jailbreak.
2. Send `nullspace-remover-ps5.elf` using a compatible ELF sender, or start it through Payload Manager.
3. Allow NULLSPACE to start its local server and register its **Media** launcher.
4. Open **NULLSPACE Game Remover** from the PS5's **Media** section. If automatic opening does not work, use the Media shortcut.
5. Select **Scan storage** to display recognized game sources.

The Media shortcut is designed to stay installed after a reboot, but **the ELF must be started again after a full restart**. Opening the shortcut without the server running will not make the remover work.

### Option B — Build from source with WSL

Install the [PS5 Payload SDK](https://github.com/ps5-payload-dev/sdk) following its official instructions. If you're using Ubuntu under WSL, install the additional basics if needed:

```bash
sudo apt update
sudo apt install -y make python3 unzip file
```

Open a terminal in the extracted NULLSPACE source folder, then run:

```bash
export PS5_PAYLOAD_SDK=/opt/ps5-payload-sdk
bash BUILD_IN_WSL.sh
```

If your SDK is installed elsewhere, replace `/opt/ps5-payload-sdk` with its actual directory. The script embeds the HTML interface and builds:

```text
nullspace-remover-ps5.elf
```

To copy the compiled ELF to your Windows Downloads folder from WSL, use:

```bash
cp nullspace-remover-ps5.elf /mnt/c/Users/<WINDOWS_USERNAME>/Downloads/
```

Replace `<WINDOWS_USERNAME>` with your Windows account name. Then load the ELF on your **already-jailbroken** PS5 using your normal sender or payload manager.

## How to use

1. **Launch NULLSPACE.** Start the ELF after jailbreaking, then open the Media app.
2. **Scan your storage.** NULLSPACE checks its recognized game-source locations and displays matching files and game folders.
3. **Find your game.** Use the artwork, title, title ID, original filename and path to identify it.
4. **Choose Manage.** Check the exact source path and whether the PS5 library link is verified.
5. **Confirm removal.** Select **Delete game** when verified library removal is available, or **Delete source** for a source-only entry.
6. **Rescan.** Check the result and confirm the unwanted game no longer appears. If its PS5 icon remains, see the section below.

**Always close and unmount the game before removing its files.** Back up anything important first.

## How deletion works

NULLSPACE treats the **original game source** and its **PS5 library entry** as separate things.

| Removal type | What NULLSPACE does |
| --- | --- |
| **Delete source** | Deletes the selected recognized game image or game folder. The PS5 library entry might remain. |
| **Delete game** | Deletes the selected source and requests uninstallation of the matching PS5 library entry **only if the source link is exactly verified**. |

For library removal, NULLSPACE checks the appropriate ShadowMount mount-link record against the **exact source path**. It does not uninstall a similarly named title or assume that a title ID in a filename proves the source link. When the link cannot be verified, it offers source-only deletion.

If Sony's uninstall request fails, the backing source may already have been deleted while the icon remains. In that case, remove the remaining entry through PS5 Settings or a compatible ShadowMountPlus manager. NULLSPACE may also remove an exact matching entry from ShadowMount's `manual.lst`, making a backup before changing that list.

**Supported source types:** `.ffpfsc`, `.ffpfs`, `.ffpkg`, `.exfat`, `.pkg`, and recognized game folders containing `sce_sys/param.json`, in the configured scan locations. Recognition of a `.pkg` file does **not** mean NULLSPACE can uninstall an unrelated normally installed PKG application.

**Safety boundaries:** the tool checks the selected filename, exact path and scan snapshot again before deletion; it avoids following symbolic links during scanning. It does not intentionally delete saved-data folders, modify PS5 system files, or directly edit the PS5 application database. Deletion still cannot be undone, and it does not identify every possible duplicate game source.

## Artwork and game names

Game titles are read from accessible game or installed-title `param.json` metadata. When proper metadata is unavailable, NULLSPACE derives a readable label from the filename and marks it **unverified**.

Artwork is read locally from supported `icon0.png` locations. Covers may be unavailable if a game image is not mounted, its metadata cannot be read, or its artwork is missing or unsupported. NULLSPACE does not extract covers from inside `.ffpfsc` or `.pkg` containers and does not download images from the internet.

## Troubleshooting

| Problem | What to try |
| --- | --- |
| **Media app opens but the library doesn't load** | Confirm that you're already jailbroken and have started the NULLSPACE ELF in this boot session. |
| **No notification or Media shortcut** | Check `/data/nullspace_remover/startup.log` over FTP for startup or app-registration errors. |
| **`bind() failed` / `errno 48`** | Port `8877` may already be in use. Fully shut down the PS5, reboot, jailbreak and load the NULLSPACE ELF only once. |
| **Missing game cover or proper title** | The local artwork or title metadata may not be accessible. Verify the filename, path and title ID before deleting. |
| **Source deleted but the icon remains** | The installed entry may not have an exact verified link, or Sony's uninstall request may have failed. Remove the icon using PS5 Settings or a compatible manager. |
| **A game reappears** | Another source copy or scan location may exist. Check all game-image locations before deleting anything else. |

NULLSPACE uses local port **`8877`** and Media title ID **`NSRM01342`**. Don't run multiple remover instances at the same time or use a conflicting port. Avoid changing the Media app's URL without also updating the ELF and launcher metadata.

## Project structure

```text
NULLSPACE/
├── assets/
│   ├── icon0.png          # Media launcher icon
│   └── param.json         # Media app metadata
├── src/
│   ├── remover.c          # Scanner, local HTTP server and removal engine
│   ├── ps5_app.c          # PS5 notifications and Media app integration
│   ├── ps5_app.h
│   └── ui_embedded.h      # Generated embedded interface
├── web/
│   └── index.html         # Offline frontend
├── tests/
│   └── test_host.py       # Desktop tests
├── embed_ui.py
├── BUILD_IN_WSL.sh
├── Makefile
├── Makefile.host
├── LICENSE
└── README.md
```

### Run desktop tests

These tests exercise the desktop simulation, **not** actual PS5 application installation or Sony's uninstall service.

```bash
make -f Makefile.host clean all
python3 -m unittest discover -s tests -v
```

## Credits

NULLSPACE is an independent homebrew project. It builds on ideas and publicly available development resources from the PS5 homebrew community, including:

- [PS5 Payload SDK](https://github.com/ps5-payload-dev/sdk) — payload development toolchain.
- [WebKit Autoloader](https://github.com/itsPLK/ps5-webkit-autoloader) — reference for PS5 Media-shortcut installation and browser integration.
- [ShadowMountPlus](https://github.com/drakmor/ShadowMountPlus) — game-mounting ecosystem and source-link conventions.

Thanks to the researchers and homebrew developers who make PS5 development possible. NULLSPACE is **not affiliated with Sony, PlayStation or the projects above**.

## License

Licensed under the [MIT License](LICENSE). See `LICENSE` for the full terms.

---

<div align="center">

**NULLSPACE v0.5** · Built for control over your own game storage.

</div>
