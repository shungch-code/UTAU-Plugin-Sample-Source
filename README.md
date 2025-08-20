# UTAU-Plugin-Sample-Source
Sample UTAU Plugin Source (2008, Ameya/Ayame, modified for research use)
## Overview
This repository contains a modified version of the original **UTAU sample plugin** by Ameya/Ayame (2008).  
It demonstrates the essential mechanism of UTAU plugins:
- Reading selected notes from a UST file
- Displaying all available parameters in a dialog **Edit Box**
- Allowing the user to edit parameters directly
- Writing changes back to UTAU so they immediately reflect in the Piano Roll
This project serves as a minimal, educational reference for researchers and developers interested in extending UTAU through plugins.  
It is also released as supplementary material for the paper:
**"Expressive Singing Enthusiasm Synthesis: Based on Open-Source UTAU Plugin Development."**
## Requirements
- Windows XP / Windows 7  
- Microsoft Visual C++ 6.0 (tested build environment)  
- UTAU (version 0.4.18 or later)
## How to Compile
1. Open `utauplugin.dsw` in **Microsoft Visual C++ 6.0**.  
2. Select `Build > Rebuild All`.  
3. The compiled executable `utauplugin.exe` will be generated in the project folder.
## How to Use
1. Copy the compiled `utauplugin.exe` into UTAU's `plugins` folder.  
2. Open UTAU and load a `.ust` file.  
3. Select one or more notes, then run the plugin from the **Plugins** menu.  
4. A dialog box will appear, showing all available parameters in an editable text box.  
5. Modify any parameters and press **OK** to apply changes back to UTAU.
## Example
**Before running (UST excerpt):**
[#0001]
Length=480
Lyric=a
NoteNum=60
**After editing in the plugin:**
[#0001]
Length=480
Lyric=a
NoteNum=60
PBW=96,96,96
PBS=0;2
---
## Credits
- Original plugin framework by **Ameya/Ayame (2008)**  
- Modified and translated for research purposes by **Shung-Che Shen (2025)**  
## License
This source code is provided for **research and educational use**.  
Further modifications and academic applications are welcome.
