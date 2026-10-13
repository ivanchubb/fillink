---
name: pdf-to-fillink
description: Convert a supplied fillable PDF into one self-contained Fillink HTML template plus a synthetic JSON submission, using the shared Fillink app for rendering, validation, saving, bulk CSV, and print workflows. Use when the user provides a PDF form and asks to add a form to Fillink.
---

# PDF to Fillink

Convert the supplied PDF into one versioned, self-contained HTML template and a synthetic JSON submission. Use the existing Fillink app for common behavior; keep templates simple and free of JavaScript. Work in the existing Fillink repository. Treat the PDF as the authoritative source for labels, wording, layout, and existing fields unless the user explicitly requests a semantic change.

## 1. Locate and inspect the source PDF

1. Use the exact file or URL the user provided. In WSL, a Windows path such as `C:\Users\name\Downloads` is usually under `/mnt/c/Users/name/Downloads`. Search that directory for the requested filename; do not scan unrelated personal files or open similarly named signed/completed forms when a blank source is available.
2. If a URL is unreachable, look for the user's stated local copy. If neither is available, ask the user to upload/copy the PDF. Do not invent the form from memory or imply a guessed conversion is faithful.
3. Inspect metadata, page count/geometry, extracted text, AcroForm/XFA widgets, field names/types/values/options, annotation rectangles, and hyperlinks. Useful tools include `pdfinfo`, `pdftotext`, `pypdf`, and PyMuPDF (`fitz`). Check tool availability rather than assuming. If a Python library is missing, install it into a temporary target/venv (for example, `pip --target /tmp/...`); do not alter the system Python environment.
4. Render every page to an image and visually inspect it. Use extracted text plus the rendered page and widget rectangles; text extraction alone loses layout, grouping, and control semantics.
5. Record the source title, edition/revision date, number of pages, and any unusual controls. Never copy personal data from an individual's completed form into examples; use conspicuously fictional data.

## 2. Map the PDF accurately

Build a field inventory before coding: visible label/instructions, source widget type, intended data path, allowed values, required/conditional rules, and any adaptation from the PDF.

- Preserve official wording and option labels verbatim where practical. Do not paraphrase attestations, privacy language, or instructions.
- Infer semantics from both labels and PDF widgets. For example, a PDF may encode separate checkbox widgets for a “Select Only One” group; represent that as HTML radios because the form explicitly requires one choice.
- Use text inputs for source free-text fields unless an enumerated choice is actually specified. Do not invent dropdown options or turn free text into a checkbox without the user's approval.
- If the user requests a semantic improvement that differs from the PDF, implement it, but clearly record the difference in the form notes. Example: the DA Form 5960 source uses text widgets for physical/legal custody; the user requested boolean checkboxes in the HTML conversion.
- Keep small fields where the PDF puts them. Initials fields should be real, bound text inputs; keep the “Read and Initial:” wording inline with each statement and put the small input in the original left-side position.
- For tables that number rows, account for the row-number column in the header (often an empty first header cell) so labels align over their data columns.
- Preserve hyperlinks as actual safe anchors; do not turn plain text into a link without source evidence.
- Treat PDF signature widgets as a signing requirement, not decoration. A typed name, date, canvas scribble, or `locked: true` is not a CAC/PIV digital signature. If cryptographic signing is not implemented, state that plainly and never imply equivalence.

If a field's meaning, valid values, or conditional requirements cannot be established from the PDF, preserve it as a text field or ask a focused question; do not silently guess.

## 3. Create one HTML template and one JSON example

- Add `templates/<form-slug>.html`: one self-contained HTML file containing accessible form markup, embedded form-specific presentation CSS, template ID/version metadata, and controls bound through `data-fillink="nested.path"`.
- Add `examples/<form-slug>.json`: one synthetic submission using the current envelope. Never use personal data copied from a completed form.
- Add `docs/forms/<form-slug>.md`: source PDF provenance, field mapping and adaptations, privacy warning, limitations, and verification status.
- Register the trusted template ID/version and local HTML path in `app/app.js`. The app fetches only this HTML file when rendering a submission. It must not trust a URI or execute code from the submission.
- Do not add a per-form manifest, JSON Schema, stylesheet, runtime script, or bulk helper. Common styling and workflows belong to Fillink. Templates must not load scripts, external stylesheets, or other assets. Keep calculations and conditional rules in native HTML constraints and declarative data attributes supported by the app; if a needed behavior is unsupported, extend the shared app rather than adding template-specific JavaScript.
- Document that the HTML conversion is unofficial unless formally approved. Keep the form notice and app-level limitations clear.

Use current envelope fields: `formatVersion`, unique `id`, `createdAt`, `template.uri`, `template.version`, required boolean `locked`, and `data`. New submissions start with `locked: false`. If `locked` is true, compatible Fillink clients must make bound fields read-only. This is only an application instruction until cryptographic integrity/signing is implemented.

Keep the same binding paths in HTML, CSV columns, and the example JSON. Use native controls where possible:

- text/date input for scalar text;
- radio group for exactly-one choices;
- boolean checkbox for a single true/false field;
- repeated checkbox group or `<select multiple>` for multi-value fields;
- `<select>` for genuinely enumerated scalar choices.

The serializer must preserve strings as strings, checkboxes as booleans, multiple selections as arrays, and repeated rows as predictable nested arrays. When loading a draft, treat omitted bindings as blank/unchecked defaults; do not reject a submission solely because the user has not filled every field. Validate every value that is present (including select values and formats) before populating so malformed data is not silently discarded. Keep completeness/required-field checks for finalization or other explicit submit workflows, not draft restoration.

## 4. Add the user workflow

### Single form

Provide simple, consistent **Save**, **Load**, **Print**, and **Lock fields** actions. Call the app **Fillink**, not “Fillink Viewer”; the same app is the primary editor. Save creates a JSON file. Load accepts a compatible JSON submission and restores values plus its locked state. Lock fields sets `locked: true` and disables bound controls; the same button should become **Unlock fields** so the user can explicitly resume editing, then Save to persist `locked: false`. Clearly state that this flag is an application convenience, not a security boundary or signature.

The local Fillink app should accept a JSON file, resolve only a trusted template ID/version allowlist, and open the filled template in a new tab so it looks like the template loaded natively. Use origin- and source-checked `postMessage` with a readiness/acknowledgement handshake. Never fetch/execute an arbitrary URI supplied by untrusted JSON. Include a popup-blocked fallback. Keep submission data in the browser and make no network upload for local viewing.

For local development bind the static server to loopback, e.g. `python3 -m http.server 8000 --bind 127.0.0.1`; do not expose a development server to the LAN. A hosting URL is not permission to upload sensitive form data.

### Bulk CSV

- Generate CSV headers from unique `data-fillink` paths in DOM order, plus a clearly marked example row derived from the template controls. The importer must skip the untouched reference row so it cannot accidentally create a synthetic submission.
- Load detects JSON vs CSV by extension/type. One populated CSV row creates one JSON submission; a successful CSV load produces a ZIP of all submissions.
- Use RFC 4180-compatible quoting and UTF-8/BOM handling for Excel. Document the cell representation of choices, booleans, arrays, and dates. For DoD IDs, instruct Excel users to import the column as **Text** so leading zeroes survive.
- Require the exact expected headers (allow reordering, reject duplicates/missing/unknown columns), skip entirely blank rows, impose a reasonable row limit, and validate every row before creating any ZIP. If one row fails, create no partial ZIP and report the CSV record number and reason.
- ZIP entries should be readable and safe. Sanitize names and avoid path separators. Use `<FORMCODE>-<sanitized-name>.json` (derive a compact stable code from the official form designator, such as `DA705AFT`); if sanitized names collide, append `-2`, `-3`, etc. Preserve the established `DA5960-<sanitized-name>.json` convention for DA Form 5960. Never put a full DoD ID or other sensitive identifier in a filename. Bulk submissions start unlocked.

## 5. Match print behavior

Create print-specific CSS rather than relying on screen layout. Match the source page size and orientation; compact margins, typography, cell spacing, and signature areas without making text illegible. Keep the form footer fixed/repeated in print when requested. Avoid page-break rules that move whole long sections to page two and create a mostly blank first page. Check Print Preview/browser output when a browser is available; if not, state that pagination remains unverified rather than claiming one-page parity.

## 6. Validate before finishing

Run checks appropriate to the repo and report their limits:

- parse every JSON file and validate example submissions against the envelope format and the selected template's bindings;
- `node --check` all JavaScript;
- parse HTML and verify app asset references resolve, while templates contain no scripts or external asset references;
- test CSV quoting, multiline cells, BOM, header validation, row errors, and blank rows;
- test ZIP integrity with `unzip -t` and inspect entry names/content;
- verify radio/checkbox/select load and save round trips, sparse/partially filled JSON draft restoration, locked submissions read-only behavior, and the new-tab handshake;
- visually inspect the HTML against every source PDF page and check print pagination when possible.

Do not claim cryptographic signing, CAC/PIV interoperability, official acceptance, or tamper evidence unless actually implemented and independently verified. Make limitations visible in the form notes and final response.

## Common mistakes to avoid

- Guessing PDF contents when a supplied URL failed; check the exact local path, otherwise ask for the file.
- Relying only on extracted text; render and inspect pages and widget positions.
- Treating every PDF button as a checkbox; respect “Select Only One” semantics.
- Silently changing official text or field meaning; document user-approved adaptations.
- Misaligning a table header by forgetting the row-number column.
- Making an initials prompt plain text without its small fillable field in the correct location.
- Letting CSS shrink the form onto two pages or leaving a normal-flow footer only at the end; use print rules and test preview. Shared base styles may use a fixed footer; override that for multi-page forms or the final page number can be repeated on every printed page. Repeat page headers/privacy blocks wherever the source PDF repeats them.
- Calling the product “Viewer,” exposing full DoD IDs in ZIP names, or labeling a browser download as “Download” when the user-facing action should be “Save.” Derive ZIP filename prefixes from the form being converted, rather than copying another template's code.
- Adding a Git SHA/hash to the footer without a stable build source; do not add hash machinery unless requested and well-defined.
- Treating `locked: true` as a cryptographic lock or signature. Until signing exists, it only tells the app to disable editing.
- Fetching arbitrary HTML from a template URI in untrusted JSON, or putting sensitive submission data in a URL, analytics, logs, or third-party service.
- Claiming the conversion is an official or accepted replacement for the PDF without approval from the form owner/records authority.
