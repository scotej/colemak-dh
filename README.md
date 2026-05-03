# Colemak-DH Windows Keyboard Layout

A custom Windows keyboard layout definition implementing the [Colemak-DH](https://colemakmods.github.io/mod-dh/) ergonomic layout variant, for use with [Microsoft Keyboard Layout Creator (MSKLC)](https://www.microsoft.com/en-us/download/details.aspx?id=102134).

## Layout

```
q w f p b  j l u y ;
a r s t g  m n e i o
x c d v z  k h , . /
```

Number row, punctuation, and all non-alpha keys remain in their standard US positions.

## Quick install

Run `installer/setup.exe` (admin required), then add the layout via **Settings > Time & Language > Language > Keyboard**. The prebuilt installer covers x86, x64, IA64, and WOW64.

## Build from source

1. Download and install [MSKLC](https://www.microsoft.com/en-us/download/details.aspx?id=102134).
2. Open `colemak dh.klc` in MSKLC.
3. Go to **Project > Build DLL and Setup Package**.
4. Run the generated `setup.exe` to install the layout.
5. Add the layout via **Settings > Time & Language > Language > Keyboard**.

## Notes

- The `.klc` file is UTF-16LE encoded, as required by MSKLC.
- VK codes follow the remapped characters (not physical key positions), so keyboard shortcuts like Ctrl+Z/X/C/V move with the letters.
- Caps Lock affects all letter keys; non-alpha keys are unaffected.
