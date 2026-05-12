# Colemak-DH Windows Keyboard Layout

A Windows implementation of the [Colemak-DH](https://colemakmods.github.io/mod-dh/) keyboard
layout. Only the letter keys are remapped; the number row, punctuation, Enter, Tab, the numpad, and all
other keys keep their standard US positions.

The layout is defined in `colemak dh.klc`, a
[Microsoft Keyboard Layout Creator (MSKLC)](https://www.microsoft.com/en-us/download/details.aspx?id=102134)
source file. A prebuilt installer is included under `installer/`, so MSKLC is not required to use the
layout. After installation the layout appears in the Windows keyboard picker as **ColemDH**.

## Contents

- [Layout](#layout)
- [Security](#security)
- [Installation](#installation)
- [Enabling the layout](#enabling-the-layout)
- [Where the files are installed](#where-the-files-are-installed)
- [Uninstalling](#uninstalling)
- [Building from source](#building-from-source)

## Layout

```
q w f p b   j l u y ;
a r s t g   m n e i o
x c d v z   k h , . /
```

- The number row, punctuation, Enter, Tab, Backspace, the numpad, and every non-letter key remain in
  their standard US positions.
- Caps Lock affects the letter keys only.
- Virtual-key codes follow the *remapped* character rather than the physical key, so shortcuts such as
  Ctrl+Z / Ctrl+X / Ctrl+C / Ctrl+V move with the letters.

## Security

**This repository has no known security risks.** The repository archive — including the prebuilt
binaries in `installer/` (`setup.exe`, the three `.msi` packages, and the per-architecture
`ColemDH.dll` files) — has been submitted to the ANY.RUN interactive malware sandbox.

The entire setup has been scanned, even though a scan is not a hollistic security defence, the script only binds certain scan codes to others, therefore the attack vector is very minimal.

- **Full report:** https://app.any.run/tasks/94e96fe3-a901-4064-b825-9ca452b80b61

Additional points:

- **Reproducible from source.** Building `colemak dh.klc` with MSKLC (*Project → Build DLL and Setup
  Package*) regenerates the files in `installer/`. If you prefer not to use any prebuilt binary, skip
  `installer/` and build the layout yourself — see [Building from source](#building-from-source).
- **A keyboard-layout DLL is passive data.** `ColemDH.dll` is a scan-code-to-character lookup table, not
  general-purpose executable code, and Windows does not load it until it is registered. See
  [Where the files are installed](#where-the-files-are-installed) for exactly what the installer adds to
  your system (two DLL files and one registry key) — nothing else is modified.

## Installation

### Prebuilt installer (recommended)

1. Clone or download this repository.
2. Run `installer/setup.exe`. It requires administrator rights — installing a keyboard layout writes to
   the Windows system directory and the registry — so accept the UAC prompt, or right-click `setup.exe`
   → **Run as administrator**. Run it from within the `installer/` folder: it requires the adjacent
   `.msi` packages and the `amd64` / `i386` / `ia64` / `wow64` subdirectories.
3. There is no setup wizard; the installer completes in a few seconds. The layout is now installed but
   not yet active — see [Enabling the layout](#enabling-the-layout).

To build and install the layout from source instead, see [Building from source](#building-from-source).

### What `setup.exe` does

`setup.exe` is a bootstrapper, not the layout itself. When run, it:

1. detects the system architecture and runs the matching package — `ColemDH_amd64.msi`,
   `ColemDH_i386.msi`, or `ColemDH_ia64.msi`;
2. that package copies the layout DLL(s) into the Windows system directory (see
   [Where the files are installed](#where-the-files-are-installed));
3. writes a registry entry that registers `ColemDH` as a keyboard layout; and
4. registers an uninstall entry under **Settings → Apps → Installed apps**.

No sign-out or restart is required; running applications pick up the layout on their next keyboard
focus.

## Enabling the layout

Installation makes the layout available; you must then select it:

1. Open **Settings → Time & language → Language & region**.
2. Next to your language (e.g. *English (United States)*), choose **⋯ → Language options**.
3. Under **Keyboards**, select **Add a keyboard → ColemDH**.
4. Switch layouts with **Win + Space** (or the language indicator in the taskbar): select **ColemDH**
   for Colemak-DH, **US** for QWERTY.

To make ColemDH the default, remove the **US** keyboard from the list, or set ColemDH as the default
input method under **Advanced keyboard settings**.

## Where the files are installed

You do not copy `ColemDH.dll` manually; `setup.exe` installs the build matching your CPU architecture.

| File in repository | Build | Installed to |
|---|---|---|
| `installer/amd64/ColemDH.dll` | 64-bit | `C:\Windows\System32\ColemDH.dll` (64-bit Windows) |
| `installer/wow64/ColemDH.dll` | 32-bit, for 32-bit applications | `C:\Windows\SysWOW64\ColemDH.dll` (also installed on 64-bit Windows, alongside the 64-bit DLL) |
| `installer/i386/ColemDH.dll` | 32-bit | `C:\Windows\System32\ColemDH.dll` (32-bit Windows) |
| `installer/ia64/ColemDH.dll` | Itanium | `C:\Windows\System32\ColemDH.dll` (Itanium Windows; obsolete) |

On 64-bit Windows the 64-bit DLL is placed in `System32` and the 32-bit DLL in `SysWOW64`. This is the
standard Windows convention, despite the counter-intuitive folder names.

`setup.exe` also creates this registry key, which is what causes Windows to load the DLL:

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layouts\a0000409
    Layout File = ColemDH.dll
    Layout Text = ColemDH
```

`a0000409` is the layout ID assigned by MSKLC; the `0409` suffix is the US-English locale. Without this
key the DLL is inert.

### Manual installation

If you have only a `ColemDH.dll` (for example, built directly in MSKLC, or copied from
`installer/<arch>/`) and do not want to run `setup.exe`, install it from an elevated PowerShell, run
from the repository root:

```powershell
# 64-bit Windows:
Copy-Item installer\amd64\ColemDH.dll  "$env:windir\System32\ColemDH.dll"
Copy-Item installer\wow64\ColemDH.dll  "$env:windir\SysWOW64\ColemDH.dll"

# Register the layout:
$k = 'HKLM:\SYSTEM\CurrentControlSet\Control\Keyboard Layouts\a0000409'
New-Item $k -Force | Out-Null
Set-ItemProperty $k 'Layout File' 'ColemDH.dll'
Set-ItemProperty $k 'Layout Text' 'ColemDH'
```

Sign out and back in, then add the layout as in [Enabling the layout](#enabling-the-layout). Note: the
modern *Add a keyboard* list may not display a layout that lacks the display-name values that
`setup.exe` also writes; if that occurs, run `setup.exe` instead.

## Uninstalling

- **Settings → Apps → Installed apps → ColemDH → Uninstall.** This removes the DLL(s) from `System32` /
  `SysWOW64` and deletes the registry key.
- Alternatively, run `installer/setup.exe` again; an installed layout can be removed from there.
- If you added ColemDH as a keyboard under your language, also remove it there
  (**Language options → Keyboards → ColemDH → Remove**).

## Building from source

Required only if you want to change the layout.

1. Install [Microsoft Keyboard Layout Creator (MSKLC)](https://www.microsoft.com/en-us/download/details.aspx?id=102134)
   (free; requires the .NET Framework).
2. In MSKLC: **File → Load Source File…**, open `colemak dh.klc`, and edit keys on the on-screen
   keyboard if desired.
3. **Project → Build DLL and Setup Package.** MSKLC compiles `colemak dh.klc` (via the Windows
   `kbdutool` / `cl` toolchain) into one `ColemDH.dll` per architecture and packages them as `setup.exe`
   plus the three `.msi` files — i.e. it regenerates the contents of `installer/`. MSKLC reports the
   output directory.
4. Run the generated `setup.exe` (as administrator) to install the build, then enable the layout as
   above.
5. To keep changes, copy MSKLC's output over `installer/`, keep `colemak dh.klc` in sync, and commit.

### `colemak dh.klc` notes

- The file must remain **UTF-16LE** encoded (the encoding MSKLC reads). Editing in place preserves the
  encoding; if the file is recreated, save it as *UTF-16 LE*. Inspect raw bytes with `xxd` or a
  hex-aware editor.
- Scan codes are hexadecimal; character columns are hexadecimal Unicode code points (`0061` = `a`).
- The `Cap` column is `1` for keys affected by Caps Lock (the letters) and `0` otherwise.
- Virtual-key codes are assigned to the remapped character, not the physical key — hence Ctrl shortcuts
  follow the letters.
- This build uses four shift states (`0`, `1`, `2`, `6`). The AltGr column (`6`) must be retained: on
  Windows 11 24H2, a layout without it causes `InputSwitch.dll` to crash Explorer, and the layout
  silently fails to activate. See the commit history for details.
