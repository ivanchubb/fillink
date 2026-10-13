# DA Form 5960 example

The single-file HTML template is `templates/da-form-5960.html`; the synthetic submission is `examples/da-form-5960.json`. The Fillink app is responsible for styles, data binding, saving/loading, lock state, CSV/ZIP workflows, and validation. The template has no script, stylesheet, manifest, or separate data-schema dependency; the app resolves it through its trusted allowlist and fetches only this HTML file.

This is an unofficial HTML recreation based on the January 2022 Army Publishing Directorate PDF. Use fictional data only. The template's 10-digit DoD ID rule, standardized grade choices, conditional spouse ID requirement, and custody checkboxes are represented through HTML controls and data attributes. The custody checkbox fields are a user-approved change from the PDF's text widgets.

Locking is a UI state only. **Unlock fields** re-enables editing, and Save persists the new boolean state. This is not tamper protection or a signature. CAC/PIV signing and official acceptance are not implemented.
