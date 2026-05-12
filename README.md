# Colemak-DH Windows Keyboard Layout

A custom Windows keyboard layout that implements the [Colemak-DH](https://colemakmods.github.io/mod-dh/)
ergonomic layout. The layout is defined in **`colemak dh.klc`** (a source file for
[Microsoft Keyboard Layout Creator — MSKLC](https://www.microsoft.com/en-us/download/details.aspx?id=102134)),
and this repo also ships a **prebuilt installer** under `installer/` so you don't need MSKLC just to use it.

It is built on the US-English locale, so once installed it appears in the Windows keyboard picker
alongside the standard **US** keyboard, named **ColemDH**.

> ### ✅ The prebuilt binaries are verified clean
>
> Everything under `installer/` — `setup.exe`, the `.msi` packages, and the per-architecture
> `ColemDH.dll` builds — has been analysed by the
> [**ANY.RUN** malware sandbox](https://app.any.run/tasks/94e96fe3-a901-4064-b825-9ca452b80b61).
> Verdict: **"No threats detected."**
>
> **Full report → https://app.any.run/tasks/94e96fe3-a901-4064-b825-9ca452b80b61**
>
> These files are also the unmodified output of building `colemak dh.klc` with MSKLC, so you can rebuild
> them yourself and compare (see section 5). Prefer not to trust any prebuilt binary at all? Ignore
> `installer/` and build the layout from source.

## The layout

```
q w f p b   j l u y ;
a r s t g   m n e i o
x c d v z   k h , . /
```

- The number row, all punctuation, Enter, Tab, Backspace, the numpad, etc. stay **exactly** where they
  are on a standard US keyboard — only the letter keys move.
- **Caps Lock still means Caps Lock** — it capitalises the letter keys above and nothing else.
- Application shortcuts follow the **letters**, not the physical keys, so Ctrl+C / Ctrl+V / Ctrl+Z /
  Ctrl+X are wherever `c` / `v` / `z` / `x` now live.

---

## 1. Install it (the easy way — use the prebuilt installer)

1. Clone or download this repo.
2. Open the **`installer/`** folder and run **`setup.exe`**. Windows will ask for administrator rights
   via a UAC prompt — say **Yes** (installing a keyboard layout writes to the Windows system folder and
   the registry). If double-clicking doesn't prompt, right-click `setup.exe` → **Run as administrator**.
   *Run it from inside `installer/`* — it needs the `.msi` files and the `amd64` / `i386` / `ia64` / `wow64`
   subfolders that sit next to it.
3. There is no wizard — it installs in a second or two and exits. The layout is now **installed** but not
   yet **active**; do section 2 below to switch to it.

### How `setup.exe` actually works

`setup.exe` is a small bootstrapper, **not** the layout itself. When you run it, it:

1. detects whether your Windows is 64‑bit, 32‑bit, or Itanium, and
2. runs the matching package — `ColemDH_amd64.msi`, `ColemDH_i386.msi`, or `ColemDH_ia64.msi` — which
3. **copies the keyboard‑layout DLL(s) into your Windows system folder** (exact paths in section 3), and
4. **writes a registry entry** that tells Windows "there's a keyboard layout called *ColemDH* in
   `ColemDH.dll`", and
5. registers an entry in **Settings → Apps → Installed apps** so you can uninstall it later.

Installing does not log you out or restart anything; running apps see the new layout the next time they
get keyboard focus. (The binaries you're running here are sandbox-verified — see the note at the top.)

---

## 2. Turn the layout on

Installing only makes the layout **available**. To actually type with it:

1. **Settings → Time & language → Language & region.**
2. Next to your language (e.g. *English (United States)*), click **⋯ → Language options**.
3. Under **Keyboards**, click **Add a keyboard** → choose **ColemDH**.
4. Switch layouts with **Win + Space** (or the language button next to the clock): pick **ColemDH** for
   Colemak-DH, pick **US** to go back to QWERTY.

To make ColemDH the default, remove the **US** keyboard from that same list, or set ColemDH as the
default input method under **Advanced keyboard settings**.

---

## 3. Where the DLL goes

**Short version: you never copy `ColemDH.dll` yourself — the installer does it for you.** This repo
ships one DLL build per CPU architecture, and `setup.exe` picks the right one(s) automatically:

| File in this repo | What it is | Installed to (by `setup.exe`) |
|---|---|---|
| `installer/amd64/ColemDH.dll` | 64‑bit layout DLL | `C:\Windows\System32\ColemDH.dll` — on 64‑bit Windows |
| `installer/wow64/ColemDH.dll` | 32‑bit layout DLL for 32‑bit apps | `C:\Windows\SysWOW64\ColemDH.dll` — on 64‑bit Windows (installed alongside the amd64 one) |
| `installer/i386/ColemDH.dll`  | 32‑bit layout DLL | `C:\Windows\System32\ColemDH.dll` — on 32‑bit Windows |
| `installer/ia64/ColemDH.dll`  | Itanium layout DLL | `C:\Windows\System32\ColemDH.dll` — on Itanium Windows (obsolete; you almost certainly don't have one) |

> Note the Windows folder names are counter‑intuitive: on 64‑bit Windows, **`System32` holds the 64‑bit
> DLL** and **`SysWOW64` holds the 32‑bit DLL**. That's normal — `setup.exe` handles it.

`setup.exe` also writes this registry key so Windows knows the DLL exists and what to call it:

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layouts\a0000409
    Layout File = ColemDH.dll
    Layout Text = ColemDH
```

(`a0000409` is the layout ID MSKLC assigned this layout; the trailing `0409` is the US‑English locale.)
The DLL on its own does nothing — Windows only loads it because of this registry entry, which is the
whole reason running `setup.exe` is the supported path.

### Installing the DLL by hand (advanced — only if you can't run `setup.exe`)

If you built just the DLL in MSKLC, or want to drop in a DLL from `installer/<arch>/` without the
installer, do it from an **elevated** PowerShell (run from the repo root):

```powershell
# On 64-bit Windows:
Copy-Item installer\amd64\ColemDH.dll  "$env:windir\System32\ColemDH.dll"
Copy-Item installer\wow64\ColemDH.dll  "$env:windir\SysWOW64\ColemDH.dll"

# Register it so Windows offers it as a keyboard layout:
$k = 'HKLM:\SYSTEM\CurrentControlSet\Control\Keyboard Layouts\a0000409'
New-Item $k -Force | Out-Null
Set-ItemProperty $k 'Layout File' 'ColemDH.dll'
Set-ItemProperty $k 'Layout Text' 'ColemDH'
```

Then sign out and back in, and add the layout as in section 2. (If it doesn't show up in the modern
*Add a keyboard* list, that's because the installer also writes display‑name values into the key — at
that point just run `setup.exe`, which sets up everything correctly.)

---

## 4. Uninstall

- **Settings → Apps → Installed apps**, find **ColemDH**, and choose **Uninstall**. This removes the
  DLL(s) from `System32` / `SysWOW64` and deletes the registry key.
- Or re-run **`installer/setup.exe`** — an already-installed layout can be removed from there.
- If you also added it as a keyboard under your language (section 2), remove it there too:
  **Language options → Keyboards → ColemDH → Remove**, so it stops appearing in the Win+Space picker.

---

## 5. Build or modify the layout yourself (MSKLC)

You only need this if you want to **change** the layout — to just use it, section 1 is enough.

1. Install [Microsoft Keyboard Layout Creator (MSKLC)](https://www.microsoft.com/en-us/download/details.aspx?id=102134)
   (free; needs the .NET Framework).
2. In MSKLC, **File → Load Source File…** and open **`colemak dh.klc`**. You'll see the layout on the
   on‑screen keyboard; click keys to edit them if you want.
3. **Project → Build DLL and Setup Package.** MSKLC compiles the `.klc` (via the Windows kbdutool / cl
   toolchain) into one `ColemDH.dll` per architecture, then bundles them into `setup.exe` + the three
   `.msi` packages — i.e. it regenerates exactly the contents of `installer/`. MSKLC tells you the
   output folder.
4. Run the generated **`setup.exe`** (as administrator) to install your build, then enable it as in
   section 2.
5. To keep your changes: copy MSKLC's output over `installer/`, keep `colemak dh.klc` in sync, and commit.

### Notes for editing `colemak dh.klc`

- It must stay **UTF‑16LE** encoded — that's what MSKLC reads. Editing in place keeps the encoding;
  if you recreate the file, save it as *UTF‑16 LE*. Inspect raw bytes with `xxd` or a hex‑aware editor.
- Scan codes are hex; the character columns are hex Unicode codepoints (`0061` = `a`).
- The `Cap` column is `1` for keys Caps Lock should affect (the letters) and `0` for everything else.
- VK codes are assigned to the **remapped** character, which is why Ctrl shortcuts move with the letters.
- This build deliberately uses 4 shift states (`0`, `1`, `2`, `6`). Don't drop the AltGr column (`6`):
  on Windows 11 24H2, a layout missing it makes `InputSwitch.dll` crash Explorer and the layout silently
  fails to activate. See the commit history for the full saga.
