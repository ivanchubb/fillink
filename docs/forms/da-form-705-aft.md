# DA Form 705-AFT example

The single-file HTML template is `templates/da-form-705-aft.html`; the synthetic submission is `examples/da-form-705-aft.json`. The Fillink app owns common styles, data binding, JSON/CSV workflows, validation, and ZIP generation. The template contains its own presentation markup and metadata, but no external script, stylesheet, manifest, or data-schema dependency. The app resolves the template by trusted ID/version and fetches only this HTML file.

Both source pages repeat the scorecard header, soldier details, scoring note, and Privacy Act statement. Grader fields are initials-only. The alternate event score is derived in the HTML declaratively: GO gives 60 points and NOGO gives 0. These calculations are simple app-supported data attributes, not per-form JavaScript.

This is an unofficial conversion of the April 2026 scorecard. It records raw values and entered points but does not calculate other AFT scores, determine eligibility, or establish official results. Soldier/OIC typed names are not digital signatures. CAC/PIV signing and official acceptance are not implemented. Use fictional data only; fitness results and names are sensitive personal information.

The Army Publishing Directorate URL could not be independently fetched during this session (HTTP 503); the local PDF was inspected. Browser print output still needs visual verification.
