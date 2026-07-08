<img width="1408" height="768" alt="miarma-logo-fondo" src="https://github.com/user-attachments/assets/030ff1ab-cbcd-4efb-a45c-f7e1e88f9ff5" />

# miARMa 0.4-beta — AYN Odin 3 (SM8750)

**miARMa** is a mutable **Arch Linux ARM** (`aarch64`) gaming image for the **AYN Odin 3**
(Qualcomm Snapdragon 8 Elite / SM8750, Adreno 830). It ships Steam (ARM64) in Game Mode,
KDE Plasma desktop, Decky Loader, FEX + Proton, pre-installed emulators, and miARMa-specific
hardware integration (rear paddles, stick LEDs, USB debugging, optional internal install).

> **Warning — prototype tester build**  
> Use at your own risk. Flashing requires a custom ABL; a failed install can brick the device
> or affect the Android partition. This image ships with default credentials (`miarma` / `miarma`).
> Change your password before enabling network services you do not trust.

**Default language:** English (Steam first-run and public documentation). KDE Plasma language
can be changed in System Settings and persists across reboots.

---

## Contents of this release

| Artifact | Description |
|----------|-------------|
| `miarma-odin3-0.4-beta.img.gz` | SD card image (flash with Balena Etcher or `dd`) |
| `abl_signed-SM8750.elf` | ROCKNIX-derived ABL for Odin 3 / SM8750 (see flashing below) |

**Primary test target:** AYN Odin 3. Other SM8750/SM8550 handhelds may boot with the same ABL
family but are **unsupported** in this beta.

---

## Quick start

1. Flash `miarma-odin3-0.4-beta.img.gz` to a **64 GB+ A2** microSD card.
2. Flash the **SM8750 ABL** to internal storage from Android (ROCKNIX-style scripts) or follow
   your device’s ABL update procedure. **Match the SoC** — wrong ABL can brick the device.
3. Boot holding **VOL−** → set device model → **Linux** boot mode → start.
4. Complete Steam first-run (language, Wi‑Fi, account). A short black screen before Steam is
   expected on first boot.
5. Optional: install to internal storage via **Desktop → miARMa Installer** (see upstream docs).

---

## SSH access over USB (PC debugging)

miARMa exposes a **USB composite gadget** with **NCM networking** (virtual Ethernet over the
data cable). SSH is **disabled by default**; enable it from the desktop shortcut **“SSH (toggle)”**
or from **miARMa Control** (Decky).

### Requirements

- USB data cable (not charge-only).
- Device awake and miARMa running (desktop or Game Mode with gadget active).
- SSH enabled (see above).

### Windows (PowerShell or Command Prompt)

1. Connect the Odin to the PC. Windows should detect a new **RNDIS/NCM** network adapter.
2. The device uses link-local addressing on the USB link:

   | Setting | Value |
   |---------|--------|
   | Device IP | `169.254.170.2` |
   | User | `miarma` |
   | Password | `miarma` (change this in production use) |
   | Port | `22` |

3. If the PC does not get an address automatically, assign the PC’s USB adapter a static
   address on the same subnet, e.g. `169.254.170.1` / mask `255.255.255.0`.
4. Connect:

   ```bash
   ssh miarma@169.254.170.2
   ```

### Linux / macOS

```bash
ssh miarma@169.254.170.2
```

If link-local ARP is stale after replug, ping once: `ping -c1 169.254.170.2`

### File access without SSH (Samba)

When the USB gadget is active, the home directory is also shared via **Samba** on the USB
interface only (not exposed on Wi‑Fi):

```text
\\169.254.170.2\home
```

Credentials: `miarma` / `miarma`

### Notes

- **Mass-storage USB disk** is **off by default** (NCM-only avoids SDP charging limits on some
  PC ports). File transfer is intended via Samba or SSH.
- Disabling SSH from the desktop shortcut **masks** `sshd` so the gadget cannot restart it
  silently on every USB attach.
- PC USB ports may negotiate **SDP (~100 mA)**; use a wall charger or powered hub for heavy
  desktop use. A **25 W** dock/charger works well for normal play.

---

## Lossless Scaling frame generation (`Lossless.dll`)

miARMa ships the open-source **lsfg-vk** Vulkan layer and the `miarma-lsfg` helper for
**Lossless Scaling–style frame generation** on **Adreno 830 (Turnip)**. The proprietary
**`Lossless.dll`** file is **not** included in this image (copyright / redistribution
restrictions). You must supply it from your own licensed copy of
[Lossless Scaling on Steam](https://store.steampowered.com/app/993090/Lossless_Scaling/).

### Where to get `Lossless.dll`

Install **Lossless Scaling** on Steam (any PC), then locate `Lossless.dll` inside the game
install directory, typically under:

```text
steamapps/common/Lossless Scaling/Lossless.dll
```

Copy that file to the Odin (USB Samba, `scp`, or microSD).

### Install on the device

**Option A — helper (recommended)**

```bash
# From SSH or Konsole on the device:
miarma-lsfg set-dll /path/to/Lossless.dll
miarma-lsfg status
```

This copies the DLL to `~/.local/share/lossless-scaling/Lossless.dll` and updates
`~/.config/lsfg-vk/conf.toml`.

**Option B — manual copy**

```bash
mkdir -p ~/.local/share/lossless-scaling
cp /path/to/Lossless.dll ~/.local/share/lossless-scaling/
miarma-lsfg status
```

On first graphical login, `miarma-lsfg-setup.service` seeds `conf.toml` if it does not exist.
The layer stays **inactive** until you enable a per-game profile.

### Enable frame-gen for a game

```bash
# Example: 2× frame-gen for a Proton game executable name
miarma-lsfg enable game.exe 2
miarma-lsfg list
```

To disable:

```bash
miarma-lsfg disable game.exe
```

Validate configuration:

```bash
miarma-lsfg validate
```

### Performance notes (Odin 3 / Turnip)

- Always use **`performance_mode=true`** (the helper sets this by default).
- Keep **`allow_fp16=false`** — FP16 is slower on Turnip in our tests.
- Full LSFG 3.1 quality mode is not usable on this GPU; performance mode is required.

---

## Tester defaults (0.4-beta)

| Item | Value |
|------|--------|
| User / password | `miarma` / `miarma` |
| `sudo` | NOPASSWD for user `miarma` |
| SSH | Off until toggled on |
| `pacman` | Mutable; critical kernel/GPU stack packages pinned in `/etc/pacman.conf` |
| SELinux | **Disabled** (standard Arch ARM; no MAC policy shipped) |

---

## Features (summary)

- Steam Game Mode (gamescope) + KDE Plasma desktop  
- Heroic (GOG/Epic), Decky Loader + **miARMa Control**, FEX + CachyOS Proton 11  
- Pre-installed emulators (ES-DE, RetroArch, Dolphin, PPSSPP, DuckStation, and others)  
- Internal **ext4** installer (`miarma-installer`) with dry-run `plan`  
- USB NCM + SSH + Samba; **exFAT** microSD support for ROMs from Windows  
- Rear paddle mapping, stick LED desktop toggle, experimental OTA path  

See `FEATURES.md` and `TODO.md` in the source repository for the full public list and roadmap.

---

## Attributions and licenses

miARMa is a distribution composed of many upstream projects. **You must comply with each
component’s license** when redistributing this image or derivatives.

### miARMa project software

Original scripts, configuration, and artwork written for miARMa are licensed under
**GPL-2.0-or-later** unless otherwise noted in the source tree.

### Major upstream components

| Component | Role | License / terms |
|-----------|------|-----------------|
| [Armada](https://github.com/virtudude/armada) | Base fork, vendor GPU stack packaging, session patterns | GPL-2.0-or-later |
| [ROCKNIX](https://github.com/ROCKNIX) | ABL, bootloader layout, DTB/kernel integration, input maps, audio profiles, firmware layout | GPL-2.0-or-later (and upstream licenses of bundled firmware) |
| [Arch Linux ARM](https://archlinuxarm.org/) | Base distribution, `pacman`, core packages | GPL and assorted (per package) |
| [Universal Blue](https://github.com/ublue-os) / [Bazzite](https://github.com/ublue-os/bazzite) / [image-template](https://github.com/ublue-os/image-template) | Image/build patterns (historical miARMa pipeline) | Apache-2.0 |
| [Steam](https://store.steampowered.com/) | Game Mode client (ARM64) | [Steam Subscriber Agreement](https://store.steampowered.com/subscriber_agreement/) — proprietary, not redistributable separately |
| [Valve gamescope](https://github.com/ValveSoftware/gamescope) / gamescope-session | Compositor session | LGPL-2.1-or-later |
| [FEX-Emu](https://github.com/FEX-Emu/FEX) | x86-64 emulation | LGPL-2.1-or-later |
| [CachyOS Proton](https://github.com/CachyOS/proton-cachyos) | Windows game compatibility layer | GPL-2.0-or-later (Proton/Wine stack; see upstream) |
| [Heroic Games Launcher](https://github.com/Heroic-Games-Launcher/HeroicGamesLauncher) | GOG/Epic launcher | GPL-3.0-or-later |
| [Decky Loader](https://github.com/SteamDeckHomebrew/decky-loader) | Quick Access Menu plugins (x86 under FEX) | GPL-2.0-or-later |
| [InputPlumber](https://github.com/ShadowBlip/InputPlumber) | Gamepad routing / capability maps | GPL-3.0-or-later |
| [KDE Plasma](https://kde.org/plasma-desktop/) | Desktop environment | GPL-2.0-or-later / LGPL |
| [lsfg-vk](https://github.com/PancakeTAS/lsfg-vk) | Vulkan frame-generation layer | GPL-3.0-or-later |
| [Lossless Scaling](https://store.steampowered.com/app/993090/Lossless_Scaling/) | **`Lossless.dll` algorithm (user-supplied only)** | **Proprietary — not shipped with miARMa** |

### Pre-installed emulators (third-party binaries)

Binaries are downloaded at image build time from upstream publishers. Each emulator remains
under its own license; miARMa does not claim ownership. Examples bundled in 0.4-beta:

| Emulator | Upstream (indicative) |
|----------|------------------------|
| ES-DE | [EmulationStation-DE](https://es-de.org/) |
| RetroArch | [Libretro](https://www.libretro.com/) |
| Dolphin | [Dolphin Emulator](https://dolphin-emu.org/) |
| PPSSPP | [PPSSPP](https://www.ppsspp.org/) |
| DuckStation | [DuckStation](https://github.com/stenzek/duckstation) |
| melonDS | [melonDS](https://melonds.kuribo64.net/) |
| Azahar, ares, Xemu, Cemu, Eden, RPCS3, AetherSX2 | Respective upstream projects / AppImage publishers (see build manifest) |

**Redistribution:** only redistribute emulator binaries if your use complies with each
upstream’s license and terms. When in doubt, link users to official download pages instead of
re-hosting.

### Firmware and vendor blobs

- **Qualcomm / device firmware** (Wi‑Fi, Bluetooth, DSP, GPU firmware) is redistributed only
  as required for hardware operation, under Qualcomm and vendor terms bundled with ROCKNIX /
  Armada packages.
- **AYN / device-specific** configuration derives from community handheld Linux work (ROCKNIX,
  Armada).

### Trademarks

Steam, Steam Deck, Proton, KDE, Plasma, Qualcomm, Snapdragon, Adreno, AYN, Odin, Lossless
Scaling, and other names are trademarks of their respective owners. miARMa is **not**
affiliated with or endorsed by Valve, AYN, Qualcomm, or the upstream projects listed above.

---

## Support and source

- **Issue tracker / source:** *(add your public repository URL here)*  
- **Version:** `0.4-beta` (`/usr/lib/miarma/version`, `/etc/os-release`)  
- **Documentation:** `FEATURES.md`, `TODO.md` (English default; Spanish copies under `docs/es/`)

---

## Legal summary

- miARMa **does not** distribute `Lossless.dll`, Steam, or other proprietary game assets.
- You are responsible for complying with Steam’s SSA, emulator licenses, and applicable law
  in your jurisdiction when using ROMs or game backups.
- This software is provided **without warranty**; see GPL-2.0-or-later for miARMa-owned code.
