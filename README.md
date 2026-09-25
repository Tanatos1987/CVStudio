CV Studio

Desktop app for building CVs and matching cover letters in Bulgarian and English. PyQt6 interface, SQLite storage, four HTML/CSS designs rendered by Chromium, one-click translation that keeps bullets and line breaks intact.

Quick start (Windows)

Double-click install_and_run.bat.

The first run finds any 64-bit Python 3.10 or newer (3.14 included), creates .venv, installs the packages from requirements.txt, downloads the fonts, builds CVStudio.exe with PyInstaller and starts it. Expect 5-10 minutes and roughly 250 MB of downloads. Later runs start the finished .exe immediately.

Command	What it does
install_and_run.bat	Build once, then just launch
install_and_run.bat rebuild	Force a fresh .exe after you edit code or templates
install_and_run.bat dev	Run from source, no build (fast when editing templates)

The finished CVStudio.exe (about 150-200 MB, Chromium is inside) runs on any 64-bit Windows 10/11 machine without Python. The first start after copying it to a new PC takes 5-10 seconds while it unpacks; after that it opens in 2-3 seconds.

Your CVs live in %APPDATA%\CVStudio\profiles.sqlite3. Back up that one file to keep every version.

Folder structure
CVStudio/
  install_and_run.bat        one-click setup, build and launch
  requirements.txt
  CVStudio.spec              PyInstaller recipe (single windowed .exe)
  main.py                    entry point
  build_tools/
    fetch_assets.py          downloads fonts (OFL), draws the app icon
  cvstudio/
    paths.py                 dev vs frozen paths, per-user data folder
    models.py                profile structure + field specs (single source of truth)
    database.py              SQLite layer with schema versioning
    translator.py            structure-preserving BG <-> EN translation
    cv_parser.py             PDF / DOCX / TXT import, photo extraction, heuristics
    renderer.py              Jinja2 engine, labels, colour palette maths
    cover_letter.py          offline cover letter draft generator
    pdf_export.py            Chromium PDF printer, optional WeasyPrint
    ui/
      theme.py               colours, QSS, fonts, dark Windows title bar
      widgets.py             bullet-aware editor, entry cards, toast
      photo.py               crop dialog and round photo frame
      pages.py               editor pages and template gallery
      preview.py             live preview panel
      import_dialog.py       old CV import review
      workers.py             background translation thread
      main_window.py         wiring
    templates/
      _shared/               base.css (fonts, A4, print), icons, page-flow macro
      modern_tech/           meta.json, style.css, header.html, cv.html, letter.html
      executive/
      creative/
      minimalist/
    assets/                  fonts/ and icon (created by fetch_assets.py)
How the main pieces work

PDF engine. WeasyPrint needs the GTK runtime on Windows, which cannot be packed into one .exe reliably. The app already embeds Chromium for the live preview, so it prints with the same engine: full flexbox and grid support, variable fonts, and the PDF is identical to the preview. WeasyPrint still works as an extra export option if you pip install weasyprint on a system where GTK is present; the menu entry appears automatically.

Multi-page CVs. Chromium in QtWebEngine ignores CSS @page margins, so each document wraps its content in _shared/flow.html, a table whose header and footer rows repeat on every printed page and create the top and bottom margins. Sidebars stay continuous on page 2 because they are painted as the root background, which Chromium draws on every sheet. The preview draws a thin red line where each new page will start.

Translation. Every multi-line field is split into bullet prefix and text. Only the text goes to Google Translate (through deep-translator), packed into as few requests as possible, then each line is put back behind its original •, - or 2.. Names and companies are transliterated with the official Bulgarian system instead of being translated, month names and "Present / Настояще" come from a dictionary, emails and links are never sent. By default the result is saved as a new version, so you keep both languages side by side.

Cover letter. Each design's letter.html includes the same header.html and style.css as its CV, so the letter carries the same name block, colours and fonts. "Write a draft from my CV" builds a first draft from your latest role, the bullets that contain numbers and your highest-rated skills. It never invents facts.

Old CV import. PDF, Word, text, and photos or scans of a CV. Scanned PDFs and images have no text layer, so they go through OCR: PyMuPDF has the Tesseract engine built in, and the Bulgarian and English language files ship in cvstudio/assets/tessdata, so nothing extra has to be installed. Expect 2-4 seconds per page. Each document is read in three orders (content stream, column-aware, top-to-bottom) and the one with the most recognised headings wins, which handles two-column layouts. The name is taken from the largest text at the top of page 1. Headings are recognised in both languages. The largest near-square image on page 1 is offered as the profile photo.

Adding your own design
Copy cvstudio/templates/minimalist to cvstudio/templates/my_design.
Edit meta.json (name, tagline, default accent, swatches, order).
Change style.css and the HTML. Available in templates: p (personal), d (all data), letter, L (labels in the chosen language), photo (data URI or empty), colour variables --accent, --accent-deep, --accent-soft, --accent-mist, --accent-ink, filters rich, initials, pretty_url, helpers contacts(p), date_range(item), split_name(name).
Run install_and_run.bat dev. The gallery thumbnail regenerates by itself when a template file changes.

Keep content inside {% call flow() %} and full-bleed decoration outside it, as the existing designs do.

Interface language

The menus are in Bulgarian by default. The БГ / EN switch next to the CV Studio name changes them; the choice is remembered. This is independent of the CV itself: the headings inside the CV follow the Design page setting and switch automatically when you translate.

Keyboard shortcuts

Ctrl+1 to Ctrl+8 switch sections, Ctrl+S saves now (it also autosaves), Ctrl+E exports the CV, Ctrl+O imports an old CV. In achievement fields, Enter continues the bullet list, Enter on an empty bullet ends it, and typing -  at the start of a line turns it into • .

Troubleshooting
Translation failed: the free Google endpoint needs internet access and sometimes rate-limits after many requests in a row. Wait a minute and retry.
Antivirus flags the .exe: single-file PyInstaller builds are sometimes flagged by heuristics. The build disables UPX to reduce this. You can also build a folder version by changing EXE(...) in CVStudio.spec to the COLLECT form from the PyInstaller docs.
Blank preview on an old laptop: start with set QTWEBENGINE_CHROMIUM_FLAGS=--disable-gpu before launching.
Diagnostics: run run_debug.bat, or open %APPDATA%\CVStudio\logs (startup.log, app.log, error.log, crash.log, qt.log).

Fonts: Manrope, Inter, Playfair Display, Unbounded and Cormorant Garamond, all under the SIL Open Font License, all with full Cyrillic.
