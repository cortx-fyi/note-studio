# Note Studio

A browser-based editor for Anki note types and card templates. Import a `.colpkg` or `.apkg` file, inspect and edit your note types, preview cards live, browse notes, and export back — all locally, no server, no account.

![Note Studio](https://img.shields.io/badge/runs%20in-browser-2563eb?style=flat-square) ![No server](https://img.shields.io/badge/no%20server-required-34d399?style=flat-square) ![iOS compatible](https://img.shields.io/badge/iOS-compatible-f59e0b?style=flat-square)

-----

## Features

**Import**

- Drag and drop or select `.colpkg`, `.apkg`, or `.anki2` files
- Supports all Anki package versions — legacy JSON schema (pre-2.1.28), protobuf schema (2.1.28+), and the latest `anki21b` zstd-compressed format
- Media files are extracted and loaded automatically, including individually compressed entries in the latest format

**Note Type Editor**

- View all note types in your deck with field and note counts
- Rename note types, add/remove/reorder fields (drag to reorder)
- Set the sort field
- Full note type ID shown (epoch ms key used by Anki internally)

**Card Template Editor**

- Separate tabs for Front template, Back template, and shared CSS
- Click any field token to insert it at the cursor position
- Supports multiple card templates per note type
- Add and delete card templates

**Live Preview**

- Enter field values manually or load from any real note in the deck
- Renders the actual Anki template syntax: `{{Field}}`, `{{FrontSide}}`, conditional blocks (`{{#Field}}` / `{{^Field}}`), and cloze deletions
- Flip between front and back
- Rendered inside a sandboxed iframe with the deck’s actual CSS applied

**Notes Browser**

- Table view of all notes for the selected note type
- Note ID column (the epoch millisecond key)
- Click any row to load that note’s field values into the preview

**Media Explorer**

- Image grid with thumbnail previews and fullscreen lightbox
- Inline audio player for sound files
- Per-file download or bulk download all

**Export**

- `models.json` — modified note type definitions
- `notes.csv` — all notes with fields as columns
- `.apkg` — repackaged deck with your edits applied, ready to import back into Anki

-----

## Usage

Open `note-studio.html` in any modern browser. No installation, no build step.

```
# Option 1: open directly
open note-studio.html

# Option 2: serve locally (avoids any file:// restrictions)
npx serve .
python3 -m http.server 8080
```

Then drag a `.colpkg` or `.apkg` onto the page, or use the Import button.

-----

## Self-hosting

The app loads four external resources at runtime. To run fully offline or avoid CDN dependencies, download these files and update the URLs in the HTML:

|File            |Source                                                    |
|----------------|----------------------------------------------------------|
|`jszip.min.js`  |cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js  |
|`sql-wasm.js`   |cdnjs.cloudflare.com/ajax/libs/sql.js/1.10.2/sql-wasm.js  |
|`sql-wasm.wasm` |cdnjs.cloudflare.com/ajax/libs/sql.js/1.10.2/sql-wasm.wasm|
|`fzstd/index.js`|cdn.jsdelivr.net/npm/fzstd@0.1.1/umd/index.js             |

`sql-wasm.wasm` is loaded separately at runtime — it must be served from the same path as `sql-wasm.js`. Update the `locateFile` callback in the HTML to point to your path.

For fonts, download the Google Fonts CSS, extract the `.woff2` URLs, host those files, and replace the `@import` with local `@font-face` declarations.

-----

## Browser compatibility

|Browser       |Status                                  |
|--------------|----------------------------------------|
|Chrome 123+   |✅ Full support (native zstd)            |
|Firefox 113+  |✅ Full support (native zstd)            |
|Safari / iOS  |✅ Supported via fzstd JS fallback       |
|Older browsers|⚠️ May work for legacy `.apkg` files only|

-----

## How it was built

The implementation was guided directly by the [Anki open-source codebase](https://github.com/ankitects/anki).

The Anki package format turned out to have three distinct generations. The oldest uses a plain SQLite database with note types stored as a JSON blob in `col.models`. A later version migrated note type definitions into separate `notetypes`, `fields`, and `templates` tables with config stored as protobuf blobs — the field numbers were read directly from `proto/anki/notetypes.proto`. The newest format (`anki21b`) wraps the database in zstd compression and replaces the JSON media manifest with a protobuf `MediaEntries` message, also with per-file zstd compression — the framing was confirmed from `rslib/src/import_export/package/`.

The protobuf decoder is a ~40-line pure JS implementation with no dependencies, handling the wire types that Anki actually uses. The SQL layer uses [sql.js](https://github.com/sql-js/sql-js) (SQLite compiled to WebAssembly) with [JSZip](https://stuk.github.io/jszip/) for archive extraction, and [fzstd](https://github.com/101arrowz/fzstd) as a pure-JS zstd fallback for iOS Safari.

-----

## License

MIT

The Anki project is licensed under AGPL-3.0. This tool reads the Anki file format but does not include or distribute any Anki source code.
