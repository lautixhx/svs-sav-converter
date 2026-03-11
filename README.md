# SVS ↔ SAV — DS Save Converter for Delta & melonDS

Transfer Nintendo DS game saves between **Delta (iOS)** and **melonDS / PKHeX** — instantly, in your browser, with no file uploads.

🔗 **[Try it live →](https://yourusername.github.io/svs-sav-converter)**

---

## The problem this solves

Delta on iOS saves DS games as **`.svs` files** — full emulator snapshots (~19 MB) that include CPU state, RAM, video buffers, and the actual game save buried inside.

melonDS on PC and tools like PKHeX expect a plain **`.sav` file** — just the raw game SRAM (usually 512 KB).

These formats are incompatible. Renaming `.svs` to `.sav` doesn't work. This tool extracts or injects the save data correctly.

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
2. Drop the `.svs` into the **SVS → SAV** tab → download the `.sav`
3. Place the `.sav` next to your ROM in melonDS, with the same filename
   - Example: `Pokemon Platinum.nds` + `Pokemon Platinum.sav`
4. Open melonDS **twice** (two windows) — load the ROM in both
5. Use the in-game Union Room / Wi-Fi Club to trade between the two instances
6. Save both games in-game
7. Drop the updated `.sav` + the original `.svs` into the **SAV → SVS** tab → download
8. In Delta: long-press the game → **Import Save State** → select the new `.svs`

### How to edit your save with PKHeX

1. Extract the `.sav` as above
2. Open **PKHeX** → File → Open → select the `.sav`
3. Edit your Pokémon, items, etc.
4. File → Save → overwrite the `.sav`
5. Inject it back into the `.svs` using the **SAV → SVS** tab
6. Import into Delta

### How to import a save without a save state

If you only have a `.sav` and no original `.svs`, you can import it directly into Delta without this tool:

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
