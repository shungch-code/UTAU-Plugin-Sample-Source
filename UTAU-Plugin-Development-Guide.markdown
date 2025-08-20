# UTAU Plugin Development Guide

*Original Source: [UTAU User Mutual Help Wiki](https://w.atwiki.jp/utaou/pages/36.html) (Japanese)*  
*Translated and Adapted for Clarity*

## Introduction

This guide provides a comprehensive overview of developing plugins for UTAU, an open-source singing voice synthesis software. It covers the plugin structure, configuration files, data formats, and detailed specifications for sections and entries used in plugin development. For additional details on plugin installation and existing plugins, refer to the [UTAU Plugin Page](https://w.atwiki.jp/utaou/pages/36.html).  

**Note**: Some information may be speculative due to limited official documentation. Testing was conducted using UTAU Ver 0.2.76. Developers should verify compatibility with newer versions.

### Table of Contents
- [Basic Information](#basic-information)
  - [Plugin Structure](#plugin-structure)
  - [Format of plugin.txt](#format-of-plugintxt)
  - [Format of install.txt](#format-of-installtxt)
  - [Folder Structure](#folder-structure)
- [Data Input/Output Methods](#data-inputoutput-methods)
  - [Input](#input)
  - [Output](#output)
- [Data Format](#data-format)
- [Sections](#sections)
  - [[#SETTING]](#setting)
  - [[#Number]](#number)
  - [[#PREV], [#NEXT]](#prev-next)
  - [[#INSERT]](#insert)
  - [[#DELETE]](#delete)
  - [[#VERSION]](#version)
- [Entries](#entries)
  - [Always-Present Entries](#always-present-entries)
    - [Length: Note Length](#length-note-length)
    - [Lyric: Lyrics](#lyric-lyrics)
    - [NoteNum: Note Number](#notenum-note-number)
    - [PreUtterance: Pre-utterance](#preutterance-pre-utterance)
  - [Potentially Omitted Entries](#potentially-omitted-entries)
    - [VoiceOverlap: Voice Overlap](#voiceoverlap-voice-overlap)
    - [Intensity: Note Intensity](#intensity-note-intensity)
    - [Modulation: Pitch Modulation](#modulation-pitch-modulation)
    - [StartPoint: STP](#startpoint-stp)
    - [Envelope: Envelope](#envelope-envelope)
    - [Tempo: Tempo](#tempo-tempo)
    - [Velocity: Consonant Velocity](#velocity-consonant-velocity)
    - [Label: Label](#label-label)
    - [$direct: Direct Output](#direct-direct-output)
    - [$patch: Direct WAV File Output](#patch-direct-wav-file-output)
    - [$region: Selection Range Start](#region-selection-range-start)
    - [$region_end: Selection Range End](#region_end-selection-range-end)
  - [Read-Only Entries](#read-only-entries)
    - [@preuttr: Auto-Adjusted Pre-utterance](#preuttr-auto-adjusted-pre-utterance)
    - [@overlap: Auto-Adjusted Overlap](#overlap-auto-adjusted-overlap)
    - [@stpoint: Auto-Adjusted STP](#stpoint-auto-adjusted-stp)
    - [@filename: Filename Used for Rendering](#filename-filename-used-for-rendering)
    - [@alias: Alias](#alias-alias)
    - [@cache: Cache Filename](#cache-cache-filename)
  - [Pitch (Mode 1)](#pitch-mode-1)
    - [PBType: Pitch Bend Type](#pbtype-pitch-bend-type)
    - [Pitches: Pitch Sequence](#pitches-pitch-sequence)
    - [PBStart: Pitch Sequence Start Position](#pbstart-pitch-sequence-start-position)
  - [Pitch (Mode 2)](#pitch-mode-2)
    - [PBS: Initial Point of Pitch Curve](#pbs-initial-point-of-pitch-curve)
    - [PBW: Pitch Point Intervals](#pbw-pitch-point-intervals)
    - [PBY: Pitch Point Movement Values](#pby-pitch-point-movement-values)
    - [PBM: Pitch Curve Shape](#pbm-pitch-curve-shape)
    - [VBR: Vibrato](#vbr-vibrato)
  - [Flags](#flags)
    - [b: BRE Without Formant Correction](#b-bre-without-formant-correction)
    - [B: BRE](#b-bre)
    - [c: Low-Pass Filter (Before Formant Correction)](#c-low-pass-filter-before-formant-correction)
    - [C: Low-Pass Filter 1](#c-low-pass-filter-1)
    - [D: Low-Pass Filter 2](#d-low-pass-filter-2)
    - [E: Low-Pass Filter 3](#e-low-pass-filter-3)
    - [F: Formant Correction Frequency Range (Pitch-Based)](#f-formant-correction-frequency-range-pitch-based)
    - [g: Simple Gender Parameter](#g-simple-gender-parameter)
    - [G: Frequency Table Regeneration](#g-frequency-table-regeneration)
    - [h: Low-Pass Filter 4 (Excluding BRE)](#h-low-pass-filter-4-excluding-bre)
    - [H: Low-Pass Filter 4](#h-low-pass-filter-4)
    - [L: Formant Correction Frequency Range (Frequency-Based)](#l-formant-correction-frequency-range-frequency-based)
    - [N: No Formant Filter](#n-no-formant-filter)
    - [P: Peak Compressor Strength](#p-peak-compressor-strength)
    - [R: Parameter File Regeneration for TIPS Engine](#r-parameter-file-regeneration-for-tips-engine)
    - [t: Pitch Shift in 10-Cent Units](#t-pitch-shift-in-10-cent-units)
    - [T: Output Frequency Table as Text](#t-output-frequency-table-as-text)
    - [W: Robot Voice Generation](#w-robot-voice-generation)
    - [Y: BRE Ratio for Stretch Range](#y-bre-ratio-for-stretch-range)
    - [/: Switch to High-Speed Batch Processing Engine](#-switch-to-high-speed-batch-processing-engine)
  - [Custom Entries](#custom-entries)
- [Reference Links](#reference-links)

## Basic Information

### Plugin Structure
A UTAU plugin requires at least the following files:
- **plugin.txt**: Configuration file defining plugin settings.
- **[Executable File]**: The main plugin executable, which must handle command-line arguments.

Optionally, an **install.txt** file simplifies installation. Other files (e.g., **readme.txt**) can be included as needed.

### Format of plugin.txt
The **plugin.txt** file specifies plugin metadata and execution settings. Example:

```plaintext
name=plugintest
execute=test.exe
shell=use
ustversion=scriptversion
notes=all
```

- **name**: Plugin name displayed in the UTAU menu.
- **execute**: Name of the executable file.
- **shell**: If set to `use`, the plugin is launched with `ShellExecuteEx`, allowing non-exe files (e.g., `.jar`, `.html`, `.hta`). Otherwise, `CreateProcess` is used.
- **ustversion**: Specifies the plugin script version (introduced in UTAU 0.4.15). Valid values:
  - `1.00`: Equivalent to UTAU 0.2.76.
  - `1.10`: Mode 1 pitch sequence moved to `Pitches` entry.
  - `1.20`: `Modulation` replaces `Moduration`, Mode 1 pitch sequence moved to `PitchBend`.
  - If omitted, UTAU’s default setting is used.
- **notes**: If set to `all`, all notes are passed, ignoring the selected range (introduced in UTAU 0.4.15). If omitted, only the selected range is passed.

### Format of install.txt
The **install.txt** file is used during plugin installation to enable drag-and-drop installation of ZIP archives in UTAU. It is optional and does not affect plugin functionality. Example:

```plaintext
type=editplugin
folder=bar
contentsdir=foo
description=This is a sample plugin
```

- **type**: Set to `editplugin` for plugins.
- **folder**: Name of the folder created under the `plugins` directory (required).
- **contentsdir**: Name of the folder containing extracted files. If omitted, defaults to `folder`.
- **description**: Optional one-line plugin description.

### Folder Structure
When using **install.txt**, the plugin distribution should follow this structure:

```
whatever.uar
├── install.txt
└── foo (folder)
    ├── plugin.txt
    ├── [Executable File]
    └── Other files (if any)
```

- `.uar` is a UTAU-specific archive extension (essentially a renamed ZIP file). ZIP files are also supported.
- Replace `foo` with the folder name specified in `contentsdir` (or `folder` if `contentsdir` is omitted).
- Without **install.txt**, manual installation (e.g., copying to the `plugins` folder) is required.

After installation, the structure becomes:

```
UTAU Installation Directory
├── utau.exe
├── plugins
│   └── bar
│       ├── plugin.txt
│       ├── [Executable File]
│       └── Other files (if any)
└── Other UTAU-related files
```

## Data Input/Output Methods

### Input
UTAU outputs selected range information to a temporary file and passes its path to the plugin via command-line arguments. The plugin reads this file to access note data.

### Output
The plugin edits the temporary file and overwrites it with the modified data.

## Data Format
The temporary file uses a text format composed of **sections**.  
- **Encoding**: Shift-JIS  
- **Line Breaks**: CR+LF  
- Sections contain **entries** with detailed note information.

## Sections
Sections begin with `[#●]` and continue until the next `[#●]`. Each section typically corresponds to one note, with different types serving distinct purposes.

### [#SETTING]
- Appears at the start of the temporary file.
- Contains basic settings (read-only; changes do not affect UTAU).
- Can be omitted in output.
- Entries:
  - **Tempo**: Song tempo.
  - **VoiceDir**: Voicebank folder.
  - **CacheDir**: Cache folder.
  - **UstVersion**: Plugin temporary file version (available in UTAU 0.4+ when “Output UST files and plugin scripts in old format” is enabled).

### [#Number]
- Contains selected range information.
- Required; cannot be omitted unless the plugin operation is canceled (in which case all sections can be omitted).
- The number is arbitrary and applied to the selected range based on output order.

### [#PREV], [#NEXT]
- **[#PREV]**: Data for the note before the selected range.
- **[#NEXT]**: Data for the note after the selected range.
- Absent if no preceding/following note exists.
- Can be omitted in output; if included, changes are applied.
- Can appear anywhere in the output, not necessarily adjacent to numbered sections.

### [#INSERT]
- Used in output to add a note at the specified position.
- Treated as a numbered section, so notes are not added outside the selected range.
- If added at the end of the selected range with no following note and `Length` is unspecified, a note with length 0 may result—exercise caution.

### [#DELETE]
- Used in output to delete a note by replacing its numbered section.
- Cannot delete other section types.

### [#VERSION]
- Introduced in UTAU 0.4+.
- Present only when “Output UST files and plugin scripts in old format” is enabled.
- Currently written as `UST Version 1.20`.

## Entries
Entries can be omitted in output, interpreted by UTAU as unchanged. Unchanged notes can return only the section header. For notes inserted with `[#INSERT]`, omitted entries receive UTAU-filled values, which differ from default note settings. “Default value” refers to values used when entries are omitted in `[#INSERT]` notes.

### Always-Present Entries

#### Length: Note Length
- **Format**: `Length=integer`
- **Range**: 1–7680
- **Unit**: Ticks (quarter note = 480)
- **Default**: Length of the following note (0 if none)
- Values >7680 are readable in Ver 0.2.76 but may cause issues in older versions. GUI input limits: 15–7680.

#### Lyric: Lyrics
- **Format**: `Lyric=string`
- **Range**: All strings except those containing line breaks, section names, or entry names followed by “=”.
- **Default**: Lyric of the following note (empty if none).
- Invalid examples: `あ[#INSERT]`, `あPreUtterance=`, `あ$foo=bar`.
- Pre-existing invalid lyrics (set via GUI) remain unchanged unless modified by the plugin.

#### NoteNum: Note Number
- **Format**: `NoteNum=integer`
- **Range**: 24–107
- **Unit**: MIDI note number (C1=24, each semitone +1)
- **Default**: Note number of the following note (24 if none).
- Notes ≥108 (C8) are not displayed on the screen.
- Notes ≥120 (C9) prevent opening note properties.
- Notes ≤23 cause synthesis errors.

#### PreUtterance: Pre-utterance
- **Format**: `PreUtterance=real`
- **Range**: <60,000
- **Unit**: Milliseconds
- **Default**: Empty (original voicebank value)
- The entry is always present, but the value can be empty.

### Potentially Omitted Entries

#### VoiceOverlap: Voice Overlap
- **Format**: `VoiceOverlap=real`
- **Range**: <60,000
- **Unit**: Milliseconds
- **Default**: Empty (original voicebank value)

#### Intensity: Note Intensity
- **Format**: `Intensity=real`
- **Range**: 0–200
- **Default**: Empty (100)
- Represents peak volume: 200 = 0 dB, 100 ≈ -6 dB.
- Precision adjustable via the `P` flag.

#### Modulation: Pitch Modulation
- **Format**: `Moduration=real` or `Modulation=real`
- **Range**: -200–200
- **Unit**: Percentage
- **Default**: Empty (100)
- Both formats supported based on UTAU version/settings.
- With fresamp-based engines, values ≤-101 may produce unusual sounds.

#### StartPoint: STP
- **Format**: `StartPoint=real`
- **Range**: Unrestricted within voicebank range
- **Unit**: Milliseconds
- **Default**: Empty (0)

#### Envelope: Envelope
- **Formats**:
  1. `Envelope=p1,p2,p3,v1,v2,v3,v4`
  2. `Envelope=p1,p2,p3,v1,v2,v3,v4,%,p4`
  3. `Envelope=p1,p2,p3,v1,v2,v3,v4,,p4` (no “%” for vowel concatenation)
  4. `Envelope=p1,p2,p3,v1,v2,v3,v4,%,p4,p5`
  5. `Envelope=p1,p2,p3,v1,v2,v3,v4,%,p4,p5,v5`
- **Range**: p values unrestricted within note range and consistent with other values; v = 0–200
- **Unit**: p = milliseconds, v = percentage
- **Default**: `0,5,35,0,100,100,0,%,0,10,100`
- Only Format 1 was valid up to Ver 0.2.35. From Ver 0.2.36, Format 1 behavior changed.
- If empty, 0 is used.

#### Tempo: Tempo
- **Format**: `Tempo=real`
- **Range**: 10–512
- **Unit**: BPM
- **Default**: Tempo from `[#SETTING]`
- Sets tempo for subsequent notes.

#### Velocity: Consonant Velocity
- **Format**: `Velocity=real`
- **Range**: 0–200
- **Default**: Empty (100)

#### Label: Label
- **Format**: `Label=string`
- **Default**: Empty

#### $direct: Direct Output
- **Format**: `$direct=boolean`
- **Range**: True
- **Default**: Empty
- Outputs without resampler (Tool 2) processing; wavtool (Tool 1) applies envelope, pre-utterance, etc.
- Any value (e.g., `$direct=False`) triggers direct output (confirmed in Ver 0.2.77).

#### $patch: Direct WAV File Output
- **Format**: `$patch=filename`
- **Range**: Relative path to WAV file in the UST file’s folder
- **Default**: Empty
- Outputs without resampler processing; wavtool applies envelope, pre-utterance, etc.
- Missing WAV files are treated as rests.
- Filenames with half-width “=” or “,” cannot render properly.

#### $region: Selection Range Start
- **Format**: `$region=range_name|range_name|…`
- **Default**: Empty
- Multiple range starts are separated by “|”.
- If duplicate range names exist in a UST, the earlier one takes precedence.

#### $region_end: Selection Range End
- **Format**: `$region_end=range_name|range_name|…`
- **Default**: Empty
- Multiple range ends are separated by “|”.
- If duplicate range names exist in a UST, the earlier one takes precedence.

### Read-Only Entries
These entries are read-only and have no effect if modified. Available in UTAU Ver 0.4.15+.

#### @preuttr: Auto-Adjusted Pre-utterance
- **Format**: `@preuttr=real`
- Auto-adjusted pre-utterance value during rendering.

#### @overlap: Auto-Adjusted Overlap
- **Format**: `@overlap=real`
- Auto-adjusted overlap value during rendering.

#### @stpoint: Auto-Adjusted STP
- **Format**: `@stpoint=real`
- Auto-adjusted STP value during rendering.

#### @filename: Filename Used for Rendering
- **Format**: `@filename=filename`
- Voicebank filename used during rendering. Not displayed if audio is missing.

#### @alias: Alias
- **Format**: `@alias=alias`
- Alias applied via `prefix.map`. Defaults to the note’s lyric if absent. Not displayed if audio is missing.

#### @cache: Cache Filename
- **Format**: `@cache=file_path`
- Cache filename. Not displayed if no cache exists.
- *Note*: In Ver 0.4.18, changed to absolute paths; cache paths may be deprecated.

### Pitch (Mode 1)

#### PBType: Pitch Bend Type
- **Format**: `PBType=value`
- **Range**: `5` or `OldData`
- **Default**: `5`
- Use `5` typically; `OldData` is for early UTAU versions.

#### Pitches: Pitch Sequence
- **Formats**: `Piches=integer,integer,…`, `Pitches=integer,integer,…`, or `PitchBend=integer,integer,…`
- **Range**: -2048–2047
- **Unit**: Cents
- **Default**: 0
- Represents pitch in 5-tick steps.
- All formats supported based on UTAU version/settings.
- Output format does not affect results.
- Omitted parts treated as 0.
- Start position varies:
  - Ver 0.2.76: From pre-utterance.
  - Ver 0.4+: From `PBStart`.

#### PBStart: Pitch Sequence Start Position
- **Format**: `PBStart=real`
- **Unit**: Milliseconds
- **Default**: 0
- Introduced in Ver 0.4+. Negative values indicate a start before note onset.
- If absent, defaults to 0 ms. In older versions (e.g., Ver 0.2.76), starts at pre-utterance.

### Pitch (Mode 2)
- Maximum pitch points: 50 (limits `PBW`, `PBY`, `PBM` values).
- Details are speculative due to limited official documentation.
- Values preserved when switching to Mode 1.
- Mode 1 pitch data remains unchanged until rendering.
- No default values, as `[#INSERT]` notes lack pitch/vibrato settings.

#### PBS: Initial Point of Pitch Curve
- **Formats**: `PBS=integer` or `PBS=integer;real`
- **Range**: -200–200; -204.8–204.7
- **Unit**: Milliseconds; 10 cents
- Specifies initial point relative to note start (time; pitch movement).
- Unspecified pitch movement defaults to 0.
- Separator: “;”.

#### PBW: Pitch Point Intervals
- **Format**: `PBW=real,real,…`
- **Range**: Unrestricted, provided it does not exceed note’s end
- **Unit**: Milliseconds
- Specifies intervals from the leftmost point.

#### PBY: Pitch Point Movement Values
- **Format**: `PBY=real,real,…`
- **Range**: -204.8–204.7
- **Unit**: 10 cents
- Specifies pitch movement for points after the initial point.
- Omitted parts treated as 0.
- Starts with the second point.

#### PBM: Pitch Curve Shape
- **Format**: `PBM=string,string,…`
- **Range**: Unspecified, `s`, `r`, or `j`
- Specifies curve shape between points:
  - Curve: Unspecified
  - Straight: `s`
  - R-type: `r`
  - J-type: `j`
- Omitted parts treated as curves.
- If all curves, entry may be omitted.

#### VBR: Vibrato
- **Format**: `VBR=real,real,real,real,real,real,real,any`
- **Range**: 0–100, 64–512, 5–200, 0–100, 0–100, 0–100, 0–100, any
- **Unit**: Percentage, milliseconds, cents, percentage, percentage, percentage, percentage, none
- Parameters: Length (relative to note), period, depth, in, out, phase, height, unused.
- Values outside ranges may be input via note properties.

### Flags
- **Format**: `Flags=string`
- **Default**: Empty
- Includes undocumented flags. Refer to note properties for effects.
- Numeric flags include ranges. Listed alphabetically.

#### b: BRE Without Formant Correction
- **Range**: 0–100
- **Default**: 0

#### B: BRE
- **Range**: 0–100
- **Default**: 50

#### c: Low-Pass Filter (Before Formant Correction)
- **Range**: 0–100
- **Default**: 50

#### C: Low-Pass Filter 1
- **Range**: 0–100
- **Default**: 0

#### D: Low-Pass Filter 2
- **Range**: 0–100
- **Default**: 0

#### E: Low-Pass Filter 3
- **Range**: 0–100
- **Default**: 0

#### F: Formant Correction Frequency Range (Pitch-Based)
- **Range**: 0–unknown
- **Default**: 3

#### g: Simple Gender Parameter
- **Range**: -100–100
- **Default**: 0

#### G: Frequency Table Regeneration
- **Default**: Unspecified
- No numeric value.

#### h: Low-Pass Filter 4 (Excluding BRE)
- **Range**: 0–99
- **Default**: 0
- If ≥100 with BRE ≥1, sound is produced.

#### H: Low-Pass Filter 4
- **Range**: 0–99
- **Default**: 0

#### L: Formant Correction Frequency Range (Frequency-Based)
- **Range**: 0–unknown (likely ≤130)
- **Default**: None

#### N: No Formant Filter
- **Default**: Unspecified
- No numeric value.

#### P: Peak Compressor Strength
- **Range**: 0–100
- **Default**: 86

#### R: Parameter File Regeneration for TIPS Engine
- **Default**: Unspecified
- No numeric value.

#### t: Pitch Shift in 10-Cent Units
- **Range**: Unknown
- **Default**: 0

#### T: Output Frequency Table as Text
- **Default**: Unspecified
- No numeric value.

#### W: Robot Voice Generation
- **Default**: Unspecified
- No numeric value (likely undocumented).

#### Y: BRE Ratio for Stretch Range
- **Range**: 0–100
- **Default**: 100

#### /: Switch to High-Speed Batch Processing Engine
- Default marker for high-speed batch processing (modifiable).

### Custom Entries
- **Format**: `$entry_name=value`
- Users can define custom entries, reflected in the “Others” field if formatted correctly.
- The leading `$` must be half-width.
- Entry names exclude line breaks, half-width spaces, and half-width “/”.
- Values containing half-width “=” or “\n” lose subsequent strings. Half-width spaces are replaced with commas.
- Example: `$you=Actually, you're an idiot`

## Reference Links
- [UTAU User Mutual Help Wiki](https://w.atwiki.jp/utaou/pages/36.html)