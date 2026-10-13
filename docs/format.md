# Draft format notes (0.1.0)

This is an exploratory contract, not a stable interoperability specification. Expect breaking changes.

## One app, one HTML template, one submission

A form has one self-contained HTML file under `templates/`. It contains the form markup, its version and ID metadata, and `data-fillink` bindings. Common styles, data binding, CSV/ZIP workflows, and save/load behavior belong to the Fillink app, not each template. Templates contain no JavaScript and may not load external stylesheets or assets.

A submission is a JSON file containing a format version, unique ID, reference to the template ID and immutable version, a boolean `locked` state, and structured `data`. The app resolves that exact reference through its trusted allowlist and fetches only the corresponding HTML file. It never blindly fetches a URI supplied by untrusted JSON. The current allowlist is local and hard-coded.

The `example.org` template URIs are reserved examples; they are not hosted and must not be treated as live services. The app currently uses HTML controls and binding attributes for field metadata and validation. A separate per-form schema/manifest is not loaded.

## Submission behavior

A locked submission is rendered read-only until the user explicitly chooses **Unlock fields**. `locked` is application state only: it can be changed by editing the JSON or using the app's unlock control, and it is not tamper protection or proof of signing. Missing bindings in an incomplete draft default to blank or unchecked; unknown bindings and invalid provided values are rejected.

The template URI identifies a template but is not, by itself, proof that a retrieved template is original. Production submissions should pin an immutable template artifact, preferably with a digest. Digest algorithm and coverage still need a normative definition.

## Binding behavior to preserve

A future renderer should:

- resolve only explicitly trusted templates;
- pin exact immutable versions and provide offline use of cached templates;
- verify template integrity before use;
- bind submitted values as text, not interpret them as HTML;
- support partial drafts while validating all supplied values;
- reject or clearly report unknown data paths;
- keep template identity and signed submission data tied together in any signing workflow.

`data-fillink` and its small declarative attributes are a minimal binding convention, not a finalized templating language.

## Signing and bulk workflows

No signature structure or signing algorithm is defined. Signing design must specify canonicalization, the exact signed bytes and claims, certificate-chain and revocation validation, timestamps, and long-term evidence. It must also be checked against CAC/PIV policies and receiving systems. Do not treat the current JSON envelope as signed or tamper-evident.

The app generates CSV headers as dot-separated paths to nested JSON values and adds a reference row with example values inferred from the template's controls and field hints. The reference row is marked in the name field and ignored on import until that marker is replaced. The app validates all populated rows before creating a ZIP. Repeated groups and a fully normative CSV dialect remain open.

## Open decisions

- Template discovery, URI rules, distribution, and repository transport.
- A normative template format and digest coverage.
- Cache layout, integrity verification, and offline UX.
- Binding syntax, localization, accessibility, conditionals, and repeatable fields.
- Signature container and CAC/PIV integration.
- Durable archival output and long-term template availability.
- License, implementation language, renderer, and validation toolchain.
