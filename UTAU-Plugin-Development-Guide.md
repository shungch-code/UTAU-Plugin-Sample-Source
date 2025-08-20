# UTAU Plugin Development Method ♬
Original link (Japanese): UTAU User Support Group @ Wiki  

## Preface ♬
This article describes in detail the development methods of UTAU plugins.  

For installation methods and a list of existing plugins, please refer to the UTAU plugin page.  
Additionally, this document contains explanations of items not mentioned elsewhere. Some information may be uncertain.  

Detailed investigation was conducted using UTAU Ver. 0.2.76.  

---

## Table of Contents
- Preface  
- Basic Information  
- Plugin Structure  
- `plugin.txt` Format  
- `install.txt` Format  
- Folder Structure  
- Data Input/Output Methods  
  - Input  
  - Output  
- Data Format  
  - Sections  
  - Items (Entries)  
- Custom Entries  
- Reference Links  

---

## Basic Information ♬

### Plugin Structure ♬
At minimum, a plugin requires the following two files:  
- `plugin.txt` (configuration file)  
- [Executable file] (the plugin program)  

The executable must be able to receive command-line arguments.  

If an additional file `install.txt` is provided, plugin installation becomes easier.  
Other files may also be included (e.g., `readme.txt`).  

---

## `plugin.txt` Format ♬
Example:
```
name=plugintest
execute=test.exe
shell=use
ustversion=scriptversion
notes=all
```

- `name`: The plugin name displayed in UTAU’s menu.  
- `execute`: The executable filename.  
- `shell=use`: Launches the plugin using ShellExecuteEx (instead of CreateProcess), enabling execution of non-EXE files (jar, html, hta, etc.).  
- `ustversion`: Introduced in UTAU 0.4.15+. Defines script version:  
  - `1.00` → equivalent to UTAU 0.2.76  
  - `1.10` → pitch sequence moved to `Pitches` entry (Mode1)  
  - `1.20` → modulation renamed to `Modulation`, pitch sequence moved to `PitchBend` entry (Mode1)  
- `notes`: (UTAU 0.4.15+) If specified, passes **all notes** to the plugin regardless of selection.  

---

## `install.txt` Format ♬
Used only for installation. Example:
```
type=editplugin
folder=bar
contentsdir=foo
description=Plugin description
```
- `type`: Always `editplugin` for plugins.  
- `folder`: Plugin folder name (required).  
- `contentsdir`: Extracted folder name (defaults to `folder`).  
- `description`: Optional, one-line plugin description.  

---

## Folder Structure ♬
With `install.txt`, distribution may look like:
```
example.uar
├─ install.txt
└─ foo
   ├─ plugin.txt
   ├─ [Executable file]
   └─ Other files
```
- `.uar` is simply a `.zip` file with a renamed extension.  
- Without `install.txt`, manual installation into UTAU’s `plugins` folder is required.  

---

## Data Input/Output ♬

### Input ♬
UTAU writes selected note information into a temporary file and passes its path to the plugin via command-line arguments. The plugin reads this file.  

### Output ♬
The plugin modifies the temporary file with edited data.  

---

## Data Format ♬
The temporary file is a text file encoded in Shift-JIS, with CR+LF line breaks.  

### Sections ♬
Each `[#[name]]` section describes one note or metadata.  

- `[SETTING]` → global settings (read-only)  
- `[number]` → selected notes  
- `[PREV]`, `[NEXT]` → neighboring notes  
- `[INSERT]` → add new notes  
- `[DELETE]` → delete notes  
- `[VERSION]` → plugin UST version  

### Items (Entries) ♬
Examples include:  
- `Length` → note length (ticks)  
- `Lyric` → lyric text  
- `NoteNum` → MIDI note number  
- `PreUtterance`, `VoiceOverlap`, `Intensity`, `Modulation`, `Envelope`, etc.  
- Pitch control (`PBS`, `PBW`, `PBY`, `PBM`)  
- Vibrato (`VBR`)  
- Flags (`Flags=B, g, W, …`)  

Some entries are read-only (e.g., `@preuttr`, `@overlap`, `@filename`).  

---

## Flags ♬
Various synthesis control flags, e.g.:  
- `B` → breathiness  
- `g` → gender factor  
- `W` → robotic voice  
- `Y` → BRE ratio scaling  
- `/` → fast batch rendering mode  

---

## Custom Entries ♬
Users may define custom entries, prefixed with `$`.  
Example:
```
$custom=value
```

---

## Reference ♬
This document is based on UTAU Ver. 0.2.76 and later updates up to Ver. 0.4.18.  
Unannounced or experimental items may exist.  
