# Note Studio

A browser-based editor for Anki note types and card templates. Import a `.colpkg` or `.apkg` file, inspect and edit your note types, preview cards live, browse notes, and export back — all locally, no server, no account.

![Note Studio](https://img.shields.io/badge/runs%20in-browser-2563eb?style=flat-square) ![No server](https://img.shields.io/badge/no%20server-required-34d399?style=flat-square) ![iOS compatible](https://img.shields.io/badge/iOS-compatible-f59e0b?style=flat-square)

**[→ Live demo: cortx.fyi/note-studio.html](https://cortx.fyi/note-studio.html)**

-----

## Screenshots

<table>
  <tr>
    <td align="center" width="50%">
      <img src="screenshots/welcome.png" alt="Welcome screen with app icon and import prompt">
      <br><sub><b>Welcome screen</b> — icon, import button, and demo deck</sub>
    </td>
    <td align="center" width="50%">
      <img src="screenshots/preview.png" alt="Live card preview showing a Basic flashcard">
      <br><sub><b>Live preview</b> — card renders with actual CSS and field values</sub>
    </td>
  </tr>
  <tr>
    <td align="center" width="50%">
      <img src="screenshots/notes.png" alt="Notes browser showing a table of all notes">
      <br><sub><b>Notes browser</b> — all notes listed with field values and note IDs</sub>
    </td>
    <td align="center" width="50%">
      <img src="screenshots/media.png" alt="Media tab showing image thumbnails and JS files">
      <br><sub><b>Media explorer</b> — images, audio, and bundled JS files with download and delete</sub>
    </td>
  </tr>
  <tr>
    <td align="center" width="50%">
      <img src="screenshots/homescreen.png" alt="Note Studio icon on iPad home screen">
      <br><sub><b>iOS home screen</b> — add to home screen from Safari for a full-screen PWA experience</sub>
    </td>
    <td></td>
  </tr>
</table>


> To add screenshots: create a `screenshots/` folder in the repo root and save the images as `welcome.png`, `preview.png`, `notes.png`, `media.png`, and `homescreen.png`.

-----

## Features

**Import**

- Drag and drop or select `.colpkg`, `.apkg`, or `.anki2` files
- Supports all Anki package versions — legacy JSON schema (pre-2.1.28), protobuf schema (2.1.28+), and the latest `anki21b` zstd-compressed format
- Media files extracted and loaded automatically, including individually zstd-compressed entries in the latest format
- Verbose debug log (tap `log` in the topbar) for diagnosing import issues

**Note Type Editor**

- View all note types with field and note counts
- Click the note type title to rename inline
- Add, remove, and drag-to-reorder fields
- Set the sort field
- Note type ID shown (Anki’s internal epoch ms key)
- Standard, Cloze, and Image Occlusion types all recognised

**Card Template Editor**

- Separate tabs for Front template, Back template, and shared CSS
- Click any field token to insert it at the cursor
- Multiple card templates per note type — add and delete
- Cmd/Ctrl+S to save

**Live Preview**

- Enter field values manually or load directly from any note in the deck
- Renders `{{Field}}`, `{{FrontSide}}`, conditional blocks (`{{#Field}}` / `{{^Field}}`), and cloze deletions
- Flip between front and back
- Sandboxed iframe with the deck’s actual CSS applied
- JS-dependent templates show a clear explanation instead of a blank card

**Notes Browser**

- Table view of all notes for the selected note type
- Note ID column (epoch ms key)
- Click any row to load field values into preview

**Media Explorer**

- Upload new media files — drag and drop or click to browse, any file type
- Delete individual files
- Image grid with thumbnail previews and fullscreen lightbox
- Inline audio player for sound files
- Per-file or bulk download

**Export**

- `models.json` — modified note type definitions
- `notes.csv` — all notes with fields as columns
- `.apkg` — repackaged deck with edits applied, ready to import back into Anki

**iOS / PWA**

- Add to home screen from Safari for a full-screen app experience
- Custom icon, theme colour, and title embedded

-----

## Limitations

**Image Occlusion**
Image Occlusion note types import and display correctly, but card preview does not work. The occlusion rendering relies on `anki.imageOcclusion.setup()` — a JavaScript API injected by Anki’s native WebView at runtime. There is no equivalent in a browser. You can still view and edit field names, template HTML, and CSS, and export back to Anki.

**JS-dependent templates**
Some decks (e.g. DeckMasterPro, AnKing’s complex templates) dynamically load external `.js` files using `globalThis.ankiPlatform` or `import()` to resolve paths relative to Anki’s app bundle. These paths don’t exist in a browser context, so the preview iframe renders blank. Note Studio detects this pattern and shows an explanatory message instead. Editing fields, templates, and CSS still works; the card will render correctly once exported back into Anki.

**`[sound:file.mp3]` in preview**
Anki replaces `[sound:filename]` tags with an audio player at review time. Note Studio’s template renderer does not process this syntax — you’ll see the raw tag in preview. The audio file itself is accessible in the Media tab.

**Cloze rendering**
Cloze preview is approximate — `{{c1::text}}` shows `[...]` on the front and the revealed text on the back. Anki supports targeting specific cloze numbers per card. Note Studio always previews all cloze markers together since there is no card scheduler context.

**No sync**
Changes are in-memory only. Nothing is sent anywhere. To persist edits, export as `.apkg` and re-import into Anki.

**Export integrity**
Exporting as `.apkg` rewrites note type definitions (models JSON and/or the notetypes table). Note content, card scheduling data, and review history pass through unchanged from the original database.

-----

## Usage

Open `note-studio.html` in any modern browser. No installation, no build step.

```bash
# Option 1: open directly
open note-studio.html

# Option 2: serve locally (avoids file:// restrictions)
npx serve .
python3 -m http.server 8080
```

Then drag a `.colpkg` or `.apkg` onto the page, or use the Import button.

-----

## Self-hosting

The app loads four external resources at runtime. To run fully offline or avoid CDN dependencies, download these and update the URLs in the HTML:

|File            |Source                                                    |
|----------------|----------------------------------------------------------|
|`jszip.min.js`  |cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js  |
|`sql-wasm.js`   |cdnjs.cloudflare.com/ajax/libs/sql.js/1.10.2/sql-wasm.js  |
|`sql-wasm.wasm` |cdnjs.cloudflare.com/ajax/libs/sql.js/1.10.2/sql-wasm.wasm|
|`fzstd/index.js`|cdn.jsdelivr.net/npm/fzstd@0.1.1/umd/index.js             |

`sql-wasm.wasm` is loaded separately at runtime — it must be served from the same path as `sql-wasm.js`. Update the `locateFile` callback in the HTML accordingly.

For fonts, download the Google Fonts CSS, extract the `.woff2` URLs, host those files, and replace the `@import` with local `@font-face` declarations.

-----

## Browser compatibility

|Browser       |Status                                                           |
|--------------|-----------------------------------------------------------------|
|Chrome 123+   |✅ Full support (native zstd decompression)                       |
|Firefox 113+  |✅ Full support (native zstd decompression)                       |
|Safari / iOS  |✅ Supported via fzstd pure-JS fallback                           |
|Older browsers|⚠️ May work for legacy `.apkg` files; `anki21b` format unsupported|

-----

## How it was built

The implementation was guided directly by the [Anki open-source codebase](https://github.com/ankitects/anki).

The Anki package format has three distinct generations. The oldest stores note types as a JSON blob in `col.models`. A later version migrated definitions into separate `notetypes`, `fields`, and `templates` tables with config as protobuf blobs — field numbers read from `proto/anki/notetypes.proto`. The newest format (`anki21b`) wraps the database in zstd compression and replaces the JSON media manifest with a protobuf `MediaEntries` message, with per-file zstd compression — confirmed from `rslib/src/import_export/package/`.

The protobuf decoder is a ~40-line pure JS implementation with no dependencies. The SQL layer uses [sql.js](https://github.com/sql-js/sql-js) (SQLite compiled to WebAssembly), [JSZip](https://stuk.github.io/jszip/) for archive extraction, and [fzstd](https://github.com/101arrowz/fzstd) as a pure-JS zstd fallback for iOS Safari.

-----

## License

MIT

The Anki project is licensed under AGPL-3.0. This tool reads the Anki file format but does not include or distribute any Anki source code.
