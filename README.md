# product-description-seo

Claude Code / Codex skill: write the per-gun `description.txt` that the
[firearm-listing-import](https://github.com/xuanji86/firearm-listing-import) skill
pushes onto a GunStore-POS Serial No and on to WooCommerce.

One pass per gun: two paragraphs (this gun, then the model), a `Title:` line, the two
California answer lines when the operator has answered them, and a `Specifications`
block — saved into `<working folder>/<serial>/description.txt`. Everything the skill
knows is in `SKILL.md`.

## Install

- **Claude Code:** `ln -sfn "$PWD/product-description-seo" ~/.claude/skills/product-description-seo`
- **Codex:** `ln -sfn "$PWD/product-description-seo" ~/.codex/skills/product-description-seo`

Then `/product-description-seo workdir` once to pick the working folder (default `~/Desktop`);
`/product-description-seo import` hands the written folders to firearm-listing-import.

## Contract with the importer

The importer strips exactly three kinds of line from the file — `Title:`, `CA Legal:`,
`Compliant Service:` — and publishes every other line verbatim. The example in `SKILL.md`
is checked against the importer's real parser before each release.
