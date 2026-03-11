# Delta ↔ SAV — DS Save Converter for Delta & melonDS

Transfer Nintendo DS game saves between **Delta (iOS)** and **melonDS / PKHeX** — instantly, in your browser, with no file uploads.

🔗 **[Try it live →](https://lautixhx.github.io/svs-sav-converter/)**

---

## The problem this solves

Delta on iOS and melonDS on PC use incompatible save formats. This tool converts between them.

Delta gives you two types of files — and they behave differently:

| File | How to get it | What it's good for |
|------|--------------|-------------------|
| `.svs` | Long-press game → Export Save State | **Extraction only** (DELTA → SAV). Cannot be reliably reimported into Delta. |
| UUID file (no extension) | `Files app → Delta → Database → SaveStates` | **Full round-trip**. Extract the `.sav`, edit it, inject it back, replace the file in Delta's folder. |

### Why can't I just rename `.svs` to `.sav`?

The `.svs` file is a full emulator snapshot (~19 MB) — it contains CPU state, RAM, video buffers, and the actual save data buried inside. melonDS expects a raw `.sav` file which is just the game's SRAM (usually 512 KB). This tool extracts or injects that inner data.

### About Delta's UUID files

Delta stores all saves internally as melonDS save state files named with a UUID like `24BB0163-10B6-45CD-B0B5-010EE9994B72` — no extension. These are identical in format to `.svs` files. The UUID file is the one you need for the full round-trip because it's the file you can physically replace in Delta's folder.

> ⚠️ Delta's built-in "Import Save State" function does not work reliably. The only method that consistently works is manually replacing the UUID file.

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

1. On your iPhone, open **Files app** → `Delta → Database → SaveStates`
2. Find the UUID file for your game and copy it to your PC
3. Drop it into the **DELTA → SAV** tab → download the `.sav`
4. Place the `.sav` next to your ROM in melonDS with the same filename
   - Example: `Pokemon Platinum.nds` + `Pokemon Platinum.sav`
5. Open melonDS **twice** (two windows) — load the ROM in both
6. Use the in-game Union Room / Wi-Fi Club to trade between the two instances
7. **Save the game in-game** (not via the emulator's save state function)
8. Drop the UUID file + the updated `.sav` into the **SAV → DELTA** tab → download
9. Replace the original UUID file in `Files app → Delta → Database → SaveStates` with the downloaded file — keeping the **exact same UUID filename** (no extension)
10. Open Delta — your Pokémon will be evolved and ready

### How to edit your save with PKHeX

1. Copy the UUID file from `Files app → Delta → Database → SaveStates` to your PC
2. Drop it into the **DELTA → SAV** tab → download the `.sav`
3. Open **PKHeX** → File → Open → select the `.sav`
4. Edit your Pokémon, items, etc. → File → Save
5. Drop the UUID file + edited `.sav` into the **SAV → DELTA** tab → download
6. Replace the UUID file in Delta's folder with the downloaded file, keeping the exact same filename

### How to work with Delta's internal UUID save files

When you save normally inside a game in Delta (not a manual save state), Delta stores it as a UUID file — something like `24BB0163-10B6-45CD-B0B5-010EE9994B72` with no extension. Despite the name, it's the exact same melonDS format as a `.svs`.

**To extract and edit this save:**
1. Connect your iPhone to your Mac with a cable → open **Finder** → select your iPhone → **Files** → Delta → locate the UUID file for your game
   - Alternatively, use the **Files app** on iOS to browse Delta's folder
2. Copy the UUID file to your computer
3. Drop it directly into the **SVS → SAV** tab of this tool — no renaming needed
4. Edit with melonDS or PKHeX
5. Drop the UUID file + the updated `.sav` into the **SAV → SVS** tab → download

### How to import a `.sav` directly into Delta (no UUID file needed)

If you only have a `.sav` and no UUID file, you can try importing it directly:

In Delta → long-press the game → **Import Save Data** → select the `.sav`

Note: this method may not work reliably. If it doesn't, you'll need the UUID file method above.

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
git clone https://github.com/lautixhx/svs-sav-converter
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
