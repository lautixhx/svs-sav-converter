# SVS ↔ SAV — DS Save Converter for Delta & melonDS

Transfer Nintendo DS game saves between **Delta (iOS)** and **melonDS / PKHeX** — instantly, in your browser, with no file uploads.

🔗 **[Try it live →](https://yourusername.github.io/svs-sav-converter)**

---

## The problem this solves

Delta on iOS saves DS games as **`.svs` files** — full emulator snapshots (~19 MB) that include CPU state, RAM, video buffers, and the actual game save buried inside.

melonDS on PC and tools like PKHeX expect a plain **`.sav` file** — just the raw game SRAM (usually 512 KB).

These formats are incompatible. Renaming `.svs` to `.sav` doesn't work. This tool extracts or injects the save data correctly.

### About Delta's internal save files (UUID files)

Delta doesn't distinguish between "save states" and "save data" internally — **it stores everything as melonDS save state files**. When you save inside a game (not a manual save state), Delta writes the result to a file named with a UUID like `24BB0163-10B6-45CD-B0B5-010EE9994B72` — no extension, but identical format to a `.svs`.

You can find these files via the **Files app** on iOS (Delta's folder) or by connecting your iPhone to a Mac with a cable. This tool accepts them directly — no renaming needed. Just drop the UUID file into the SVS slot and it will work exactly the same.

---

## Use cases

| Goal | How |
|------|-----|
| Trade-evolve Pokémon with yourself | Extract `.sav` → load in melonDS → trade → inject back |
| Edit your save with PKHeX | Extract `.sav` → open in PKHeX → edit → inject back |
| Back up your Delta save as a standard file | Extract `.sav` and keep it |
| Continue on PC and return to iPhone | Extract `.sav` → play in melonDS → inject updated save back into `.svs` |

---

## How it works

### SVS → SAV (Extract)

Delta's `.svs` is a melonDS save state. Its internal structure looks like this:

```
[MELN header — 16 bytes]
[NDSG section — ~16 MB  — GPU/CPU/RAM snapshot]
[DMA sections — ~600 bytes]
[ARM9 / ARM7 sections]
[NDSC section — ~16 KB  — cartridge state]
[SRAM metadata — 24 bytes]
[SRAM data — 512 KB     ← this is your actual game save]
[WIFI section]
...
```

The tool:
1. Verifies the `MELN` magic bytes to confirm it's a valid melonDS save state
2. Scans for the `NDSC` (NDS Cartridge) section
3. Reads the SRAM size field from the metadata header right after `NDSC`
4. Extracts exactly that many bytes as a raw `.sav` file

### SAV → SVS (Inject)

The reverse operation:
1. Loads the original `.svs` (to preserve all emulator state)
2. Locates the SRAM region using the same method above
3. Overwrites those bytes with the contents of the new `.sav`
4. Outputs the modified `.svs`, same size as the original

The rest of the save state (CPU registers, screen state, etc.) is kept intact, so Delta can load it normally.

---

## Step-by-step guides

### How to trade-evolve a Pokémon (e.g. Gengar, Alakazam)

1. In **Delta**, long-press the game → **Export Save State** → save the `.svs` file
   - Or locate the UUID save file in `Files app → Delta → Database → SaveStates`
2. Drop the file into the **SVS → SAV** tab → download the `.sav`
3. Place the `.sav` next to your ROM in melonDS, with the same filename
   - Example: `Pokemon Platinum.nds` + `Pokemon Platinum.sav`
4. Open melonDS **twice** (two windows) — load the ROM in both
5. Use the in-game Union Room / Wi-Fi Club to trade between the two instances
6. Save both games in-game
7. Drop the updated `.sav` + the original file into the **SAV → SVS** tab → download
8. **To import back into Delta:**
   - Open the **Files app** on iOS → `Delta → Database → SaveStates`
   - Replace the original UUID file with the downloaded file, keeping the exact UUID filename
   - Open Delta — your Pokémon will be evolved and ready

### How to edit your save with PKHeX

1. Extract the `.sav` as above
2. Open **PKHeX** → File → Open → select the `.sav`
3. Edit your Pokémon, items, etc.
4. File → Save → overwrite the `.sav`
5. Inject it back into the `.svs` using the **SAV → SVS** tab
6. Import into Delta

### How to work with Delta's internal UUID save files

When you save normally inside a game in Delta (not a manual save state), Delta stores it as a UUID file — something like `24BB0163-10B6-45CD-B0B5-010EE9994B72` with no extension. Despite the name, it's the exact same melonDS format as a `.svs`.

**To extract and edit this save:**
1. Connect your iPhone to your Mac with a cable → open **Finder** → select your iPhone → **Files** → Delta → locate the UUID file for your game
   - Alternatively, use the **Files app** on iOS to browse Delta's folder
2. Copy the UUID file to your computer
3. Drop it directly into the **SVS → SAV** tab of this tool — no renaming needed
4. Edit with melonDS or PKHeX
5. Drop the UUID file + the updated `.sav` into the **SAV → SVS** tab → download

**To import the result back into Delta:**

> ⚠️ Delta's built-in "Import Save State" button may not work reliably. Use the manual method below instead.

1. Open the **Files app** on iOS
2. Navigate to `Delta → Database → SaveStates`
3. Find the original UUID file for your game
4. Replace it with the downloaded file — renaming it to match the original UUID exactly (no extension)
5. Open Delta — the game will load with the updated save

### How to import a `.sav` directly into Delta (no save state needed)

If you only have a `.sav` and no original `.svs` or UUID file:

In Delta → long-press the game → **Import Save Data** → select the `.sav`

---

## Supported games & SRAM sizes

The tool auto-detects SRAM size. Supported sizes:

| Size | Example games |
|------|--------------|
| 512 KB | Pokémon Platinum, HeartGold, SoulSilver, Black, White, Black 2, White 2 |
| 256 KB | Pokémon Diamond, Pearl |
| 128 KB | Many RPGs and adventure games |
| 64 KB | Various DS titles |
| 32 KB | Smaller DS games |

---

## Privacy

Everything runs locally in your browser using the JavaScript [File API](https://developer.mozilla.org/en-US/docs/Web/API/File_API). **No data is ever uploaded to any server.** You can use this tool offline once the page is loaded.

---

## Running locally

It's a single HTML file with no dependencies or build steps.

```bash
git clone https://github.com/yourusername/svs-sav-converter
cd svs-sav-converter
open index.html   # or just double-click it
```

---

## Tech notes

- The melonDS save state format starts with `MELN` magic bytes followed by a version number and total file size
- Sections are identified by 4-byte ASCII IDs (`NDSG`, `NDSC`, `ARM9`, `ARM7`, `WIFI`, etc.) followed by a 4-byte little-endian length field
- The SRAM block lives immediately after the `NDSC` section, preceded by a 24-byte metadata header containing the backup type and size
- SRAM size is read from offset `+12` of that metadata header as a little-endian uint32
- If the size field is not found, the tool falls back to heuristic detection: scanning for large regions of `0xFF` bytes (erased flash memory) at the expected location

---

## Contributing

Issues and PRs welcome. If you find a DS game whose save isn't extracted correctly, open an issue and attach the first 64 bytes of your `.svs` (use a hex editor or browser devtools) — no personal save data needed.

---

## License

MIT
