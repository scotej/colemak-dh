# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A custom Windows keyboard layout definition implementing the **Colemak-DH** ergonomic layout variant. The repository contains a single `.klc` file for use with Microsoft Keyboard Layout Creator (MSKLC).

## Key File

`colemak dh.klc` — UTF-16LE encoded Windows keyboard layout file. This is the only source file in the repo.

## KLC File Format

The `.klc` format has these sections in order:

- **Header** — `KBD`, `COPYRIGHT`, `COMPANY`, `LOCALENAME`, `LOCALEID`, `VERSION`
- **SHIFTSTATE** — defines which modifier columns appear in the LAYOUT table (column 4 = normal, 5 = Shift, 6 = Ctrl, 7 = Shift+Ctrl)
- **LAYOUT** — maps scan codes to virtual keys and Unicode codepoints per shift state. Format: `SC  VK_  Cap  col4  col5  col6  col7  // comment`. A value of `-1` means "not defined for this shift state"
- **KEYNAME / KEYNAME_EXT** — display names for keys
- **DESCRIPTIONS / LANGUAGENAMES** — locale metadata
- **ENDKBD** — end marker

## Colemak-DH Layout (alpha keys)

```
q w f p b  j l u y ;
a r s t g  m n e i o
x c d v z  k h , . /
```

## Working with the File

- The file is **UTF-16LE** encoded. Use `xxd` or a hex-aware editor to inspect raw bytes. Standard text tools may display wide-spaced characters.
- To compile into an installable Windows layout, use MSKLC (`msklc.exe`) on Windows.
- Scan codes are in hex; Unicode codepoints in columns are also hex (e.g., `0061` = 'a').
- The `Cap` column: `0` = unaffected by Caps Lock, `1` = affected by Caps Lock.
