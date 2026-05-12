# Colemak-DH for Windows

This is the [Colemak-DH](https://colemakmods.github.io/mod-dh/) keyboard layout, packaged up for
Windows. It only remaps the letter keys — your number row, punctuation, Enter, Tab, the numpad and
everything else stay exactly where they are on a normal US keyboard. Caps Lock still behaves like Caps
Lock, and shortcuts like Ctrl+C / Ctrl+V follow the letters, so they end up wherever `c` and `v` now
live.

The layout itself is `colemak dh.klc` — the source file you'd open in
[Microsoft Keyboard Layout Creator (MSKLC)](https://www.microsoft.com/en-us/download/details.aspx?id=102134) —
but you don't need MSKLC just to use it: there's a prebuilt installer in `installer/`. Once it's
installed, Windows lists the layout in its keyboard picker as **ColemDH**, right next to the usual
**US** one.

## Installing it (the easy way)

1. Clone or download this repo.
2. Open the `installer/` folder and run **`setup.exe`**. Windows pops a UAC prompt asking for admin
   rights — say yes; installing a keyboard layout has to write into the Windows system folder and the
   registry. (If a plain double-click doesn't prompt, right-click → **Run as administrator**.) Run it
   from *inside* `installer/` — it expects the `.msi` files and the `amd64` / `i386` / `ia64` / `wow64`
   folders to be sitting right next to it.
3. There's no wizard to click through — it does its thing in a second or two and quits.

After that the layout is *installed* but not *active*: Windows knows it exists, but you're still typing
QWERTY until you switch to it (next section).

### What `setup.exe` actually does

`setup.exe` is just a little launcher — the real layout is the `ColemDH.dll` it carries around. When
you run it, it:

- figures out whether your Windows is 64-bit, 32-bit or Itanium and runs the matching package
  (`ColemDH_amd64.msi`, `ColemDH_i386.msi` or `ColemDH_ia64.msi`);
- has that package copy the layout DLL into your Windows system folder (exactly where is spelled out
  below);
- adds a registry entry so Windows knows there's a keyboard layout called *ColemDH* in `ColemDH.dll`;
  and
- registers it in **Settings → Apps → Installed apps**, so you can uninstall it the normal way later.

None of that logs you out or restarts anything — open apps pick up the new layout the next time they
get keyboard focus.

If you'd rather not run a prebuilt `.exe` on faith: the files under `installer/` (`setup.exe`, the
`.msi` packages, the `ColemDH.dll` builds) have been through the
[ANY.RUN sandbox](https://app.any.run/tasks/94e96fe3-a901-4064-b825-9ca452b80b61) and nothing was
flagged. They're also just the output of building `colemak dh.klc` with MSKLC, so you can rebuild and
compare — or skip `installer/` altogether and build it yourself (see *Building it yourself* below).

## Switching to it

Installing only makes the layout *available* — you still have to pick it:

1. Open **Settings → Time & language → Language & region**.
2. Find your language (e.g. *English (United States)*), click the **⋯** next to it, and choose
   **Language options**.
3. Under **Keyboards**, hit **Add a keyboard** and pick **ColemDH**.
4. From then on, **Win + Space** cycles through your keyboards (so does the little language button down
   by the clock) — pick **ColemDH** for Colemak-DH, **US** to drop back to QWERTY.

Want ColemDH to be the default? Either remove **US** from that keyboard list, or set ColemDH as the
default input method under **Advanced keyboard settings**.

## Where the DLL ends up

Short answer: you don't put it anywhere — `setup.exe` does. But the question always comes up, so here's
exactly where things land.

The repo carries one `ColemDH.dll` build per CPU architecture, and `setup.exe` picks the right one(s)
for your machine:

| In the repo | What it is | Where `setup.exe` puts it |
|---|---|---|
| `installer/amd64/ColemDH.dll` | the 64-bit build | `C:\Windows\System32\ColemDH.dll` — on 64-bit Windows |
| `installer/wow64/ColemDH.dll` | the 32-bit build, for 32-bit apps | `C:\Windows\SysWOW64\ColemDH.dll` — also installed on 64-bit Windows, next to the one above |
| `installer/i386/ColemDH.dll` | the 32-bit build | `C:\Windows\System32\ColemDH.dll` — on 32-bit Windows |
| `installer/ia64/ColemDH.dll` | the Itanium build | `C:\Windows\System32\ColemDH.dll` — on Itanium Windows, i.e. basically nobody |

And yes, the folder names look backwards: on 64-bit Windows the *64-bit* DLL goes in `System32` and the
*32-bit* one in `SysWOW64`. That's a long-standing Windows quirk, not a mistake — `setup.exe` handles
it.

It also writes a registry key, and that's the bit that actually makes Windows notice the DLL:

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layouts\a0000409
    Layout File = ColemDH.dll
    Layout Text = ColemDH
```

(`a0000409` is the ID MSKLC assigned this layout; the `0409` on the end is the US-English locale.)
Without that key the DLL just sits on disk doing nothing — which is the whole reason `setup.exe` is the
supported route, rather than copying files around by hand.

### Doing it by hand

If all you've got is a loose `ColemDH.dll` — you built only the DLL in MSKLC, say, or pulled one out of
`installer/<arch>/` — and you don't want to use the installer, you can wire it up yourself from an
**elevated** PowerShell, run from the repo root:

```powershell
# 64-bit Windows:
Copy-Item installer\amd64\ColemDH.dll  "$env:windir\System32\ColemDH.dll"
Copy-Item installer\wow64\ColemDH.dll  "$env:windir\SysWOW64\ColemDH.dll"

# tell Windows about it:
$k = 'HKLM:\SYSTEM\CurrentControlSet\Control\Keyboard Layouts\a0000409'
New-Item $k -Force | Out-Null
Set-ItemProperty $k 'Layout File' 'ColemDH.dll'
Set-ItemProperty $k 'Layout Text' 'ColemDH'
```

Then sign out and back in and add it the same way as in *Switching to it*. One catch: the modern
*Add a keyboard* list sometimes won't show a layout that's missing the extra display-name values the
installer normally writes — if that bites you, just run `setup.exe` and let it do the full job.

## Removing it

- The tidy way: **Settings → Apps → Installed apps**, find **ColemDH**, **Uninstall**. That deletes the
  DLL(s) from `System32` / `SysWOW64` and removes the registry key.
- Or run `installer/setup.exe` again — if the layout's already installed, it offers to take it back out.
- If you also added it as a keyboard under your language, remove it there too
  (**Language options → Keyboards → ColemDH → Remove**), otherwise it keeps showing up in the Win+Space
  rotation.

## Building it yourself

You only need this if you actually want to *change* the layout — to just use it, the installer above is
everything.

1. Install [Microsoft Keyboard Layout Creator (MSKLC)](https://www.microsoft.com/en-us/download/details.aspx?id=102134).
   It's free, and it wants the .NET Framework.
2. In MSKLC, **File → Load Source File…** and open `colemak dh.klc`. The layout shows up on the
   on-screen keyboard, and you can click keys to remap them.
3. **Project → Build DLL and Setup Package.** MSKLC compiles the `.klc` (through Windows' kbdutool / cl
   toolchain) into a `ColemDH.dll` for each architecture and wraps them in a `setup.exe` plus the three
   `.msi` packages — basically regenerating everything that's in `installer/`. It tells you where it
   left the output.
4. Run that `setup.exe` (as administrator) to install your build, then switch to it as above.
5. To keep the change, copy MSKLC's output over `installer/`, keep `colemak dh.klc` in step with it,
   and commit.

A few things worth knowing if you're editing `colemak dh.klc` directly:

- It has to stay **UTF-16LE** — that's the encoding MSKLC reads. Editing it in place keeps that; if you
  ever recreate the file, save it as *UTF-16 LE*. To look at the raw bytes use `xxd` or a hex-aware
  editor (an ordinary text editor just shows it full of wide-spaced characters).
- Scan codes are in hex, and the character columns are hex Unicode codepoints — `0061` is `a`.
- The `Cap` column is `1` for the keys Caps Lock should affect (the letters) and `0` for everything
  else.
- VK codes follow the *remapped* character, not the physical key — that's why Ctrl shortcuts travel
  with the letters.
- This build keeps four shift states on purpose (`0`, `1`, `2`, `6`). Don't drop the AltGr column
  (`6`): on Windows 11 24H2 a layout without it makes `InputSwitch.dll` take Explorer down with it, and
  the layout quietly fails to activate. The commit history has the full, painful story.
