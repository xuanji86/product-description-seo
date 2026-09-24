---
name: product-description-seo
argument-hint: "[workdir [path] | import [all|A,B]]"
description: Generate WooCommerce/WordPress-ready SEO product descriptions for firearms from user-provided gun data. Use when writing or revising firearm product listings, catalog descriptions, specification blocks, collector firearm descriptions, military surplus descriptions, or SEO-friendly WordPress/WooCommerce copy where factual restraint, a per-gun listing title, California compliance answer lines, minimum word count, masked serial numbers, honest labeling of conversions and clones, hedged provenance, targeted follow-up questions, a confirmed working folder, serial-folder placement, structured specifications, and an optional hand-off to the firearm-listing-import skill are required.
---

# Product Description SEO

## Overview

Write concise, factual, SEO-friendly firearm product descriptions from the information the user provides. Preserve the user's facts, avoid unsupported claims, and use reputable model background only when needed to meet the requested length or improve search value.

Output from this skill is normally consumed by the `firearm-listing-import` skill, which reads each `description.txt` and pushes it to the ERPNext **Serial No** record and on to WooCommerce. The `Title:` line, the two California answer lines, and the exact filename below are contract requirements of that importer, not stylistic preferences. The importer strips only those three kinds of line; every other line in the file lands on the product page verbatim.

## Working Folder

The working folder is the batch folder whose subfolders are named after serial numbers. It is remembered across sessions in the plain-text file `~/.config/product-description-seo/workdir` (one line: the absolute path).

On Windows `~` is the user's home folder (e.g. `C:\Users\<name>`), so the default is `C:\Users\<name>\Desktop`. Always save the expanded absolute path, never a literal `~`, `%USERPROFILE%` or `$HOME`.

- **First run** (the file does not exist): before researching or writing, ask the user where the working folder is. Offer the Desktop (`~/Desktop`) as the default and accept an absolute path or a folder name under the Desktop. Expand `~`, check the folder exists, save the path to the file, and confirm it in one line.
- **Later runs**: read the file, state the folder in one line at the start (`Working folder: ~/Desktop/OSA batch 3`) and carry on without asking. If the saved folder no longer exists, say so and ask again as on the first run.
- A batch folder the user names in the request wins for that run without changing the saved default.

Every `description.txt` is saved under the working folder; never guess a location when the folder has not been confirmed.

## Commands

The skill can be invoked directly as `/product-description-seo` with an optional argument (`$ARGUMENTS`):

- `/product-description-seo workdir` — show the current working folder and offer a short menu: keep it, use the Desktop, or enter another path. Save the choice to the file above and confirm.
- `/product-description-seo workdir <path>` — set the working folder to `<path>` (absolute, `~`-prefixed, or a folder name under the Desktop) without a menu. Refuse a path that does not exist and say which path was tried.
- `/product-description-seo import` — hand the folders written in this session to the `firearm-listing-import` skill (see Handing Off to the Importer). With nothing written this session, say so and stop.
- `/product-description-seo import <A,B>` — import exactly those serial folders (names as on disk, comma-separated); `/product-description-seo import all` — every serial folder under the working folder.
- `/product-description-seo` with anything else, or nothing — the normal listing workflow, starting with the working-folder check above.

Only the working folder is stored; no gun data, credentials or descriptions go into the config file.

## Workflow

One gun, one pass. The draft is written and saved in the first reply; questions ride along behind it and never hold it up.

1. **Working folder** — confirm or read it (see Working Folder).
2. **Facts** — take the operator's text as the primary source. If the serial folder already exists and holds photos, look at `main.*` and the other photos before writing: markings, finish, furniture and accessories visible in them may be used, the operator's wording wins where the two differ, and a real contradiction is raised as a question. While looking, note two things the importer will enforce later and say them now rather than at import time: the primary photo (`main.*`, else the first by filename) must be wider than tall, and the gun in it must be the right way up (barrel level, sights on top). A sideways or upside-down primary is fixed on the spot when the `firearm-listing-import` skill is available: its `rotate` command turns one photo clockwise by 90, 180 or 270 degrees in place and keeps the original as `.orig` (`uv run scripts/firearm_listings.py rotate --root "<working folder>" --folder <serial> --file main.jpg --degrees 90`), and you look again afterwards. Without that skill, or when no turn can fix it (mirrored, wrong crop), leave a line in the chat reply: `Photo note: main.jpg is portrait — the importer will refuse it; rotate or pick another main.*`. No photos, no folder: skip this step, it is optional.
3. **Quick model check** — a short look at the model (opened source, not memory) to catch what the copy must get right: conversion, clone or reproduction status, multiple factory chamberings, importer and import history, arsenal or manufacturer naming. This is what feeds the background paragraph and the authenticity labels.
4. **Draft and save** — write the whole listing (California lines if already answered, `Title:`, two paragraphs, `Specifications`), save it as `description.txt` in the serial folder, and show it in the same reply. Below the listing, in the chat only: the `Sources:` line, then `Questions to improve the listing`, which always includes `CA Legal` and `Compliant Service` while they are unanswered.
5. **Fold answers back in** — when the operator answers any question or corrects a fact, in this batch or later, rewrite that gun's `description.txt` at once and report one line: `Updated <folder>/description.txt`. Rewrites inside the same batch are expected and need no further permission.
6. **Close the batch** — counts of files written and folders created, then the `Import:` hint (see Handing Off to the Importer).

## Core Rules

- Do not invent firearm details, provenance, history, condition, rarity, matching status, accessories, measurements, import marks, bore condition, mechanical status, or features.
- State the caliber or gauge when it is inherent to the model, even if the user did not supply it: a K98k is 8mm Mauser, a Type 38 is 6.5x50mm, a Mosin Nagant is 7.62x54R, an M1 Garand is .30-06. This is the one specification that may be filled in from the model designation. Ask the user to confirm only when the model was chambered in more than one cartridge, such as a vz.24 or a Mauser Standard-Modell.
- Preserve exactly, without paraphrase or upgrade: markings, proof marks, import marks, matching status, finish notes, provenance details, and any acronym expansion the user asks for, such as "RHKP is Royal Hong Kong Police."
- Never print a full serial number in the visible description. Mask the last two digits with `xx`, so 12345 becomes 123xx and 5711517 becomes 57115xx. The full serial is still used for research and for locating or naming the save folder; only the customer-facing listing text is masked.
- The prose is two paragraphs, written in one pass and totalling roughly 180 to 250 words. The `Title:` line and the `Specifications` block do not count.
  - **Paragraph 1, this gun** (about 60 to 100 words): only the operator's facts and what the photos show, in catalog order: what it is, chambering, year and maker, configuration and conversions, condition, bore and mechanics, what comes with it.
  - **Paragraph 2, the model** (about 100 to 150 words): conservative, model-specific background from an opened source, kept separate from the item facts. Write it by default; do not ask permission for it. It may close with one sentence that places this gun against that history (`A 2003 rifle predates that ban by eleven years.`).
- Never pad with invented detail. When the operator's facts are genuinely thin, paragraph 1 stays short and the reply says so in chat; paragraph 2 still carries the listing to length.
- No price anywhere in the description. The price is a POS field pushed separately, and text goes stale.
- The title never carries a serial number, masked or not.
- Avoid sales puffery that implies unsupported performance, investment value, combat use, historical attribution, or guaranteed collectibility. Do not call an item rare, desirable, or a good investment.
- Restraint about scarcity means silence, not the opposite claim. Never write that an item is common, plentiful, widely available, "circulates widely," or words to that effect. The rule above bars claiming scarcity; it does not license asserting abundance. Where a variant is objectively less often encountered than the standard configuration, that comparison may be stated as fact.
- Use firearm terms naturally for SEO, such as manufacturer, model, caliber/gauge, action, condition, bore, finish, stock, matching, military surplus rifle, collector firearm, or shotgun, when those terms fit the supplied facts.
- Do not provide legal, gunsmithing, modification, ammunition-loading, or usage instructions.
- If a fact is uncertain from the input, either omit it or label it as the user's wording rather than converting it into a stronger claim.
- Physical features the operator reports are facts, not claims to verify. The operator has the firearm in hand; a reference describes the type, the operator is describing this object. Dimensions, thread pitches, markings, finishes, part types, counts and configuration go into the copy as reported. Do not contradict them with general sources about the model, and do not raise an objection the operator's own description already answered. Research is for background, history and context, never for second-guessing an observable feature. Question a supplied fact only when it is internally impossible, such as a receiver type that the model never had, a date preceding the model's existence, a serial outside a documented range, or a description that contradicts the photos in the folder.
- Normalize shorthand and slang in the `Specifications` block to the formal name for search value, for example "Izzy" to "Izhevsk" and "Tula arsenal" to "Tula". The user's own phrasing may stay in the prose.

## Authenticity and Provenance

Proactively flag and clearly label conversions, clones, and reproductions. Never let copy imply a firearm is factory-original when it is not. Word the disclosure so the listing stays accurate and defensible on value rather than apologetic.

Recurring cases:

- Golden State or "Santa Fe" Jungle Carbine: a commercial No.4 conversion, not a genuine No.5 Mk I.
- K98 ZF4 "swept-back" sniper: a clone of the rare original, not a factory sniper.
- PU Mosin Nagant: a professionally executed PU-configuration conversion, not an original PU sniper.
- Yugoslav M59 versus M59/66: the base M59 has no grenade launcher. Distinguish the two, and distinguish a TRB-refurbished M59 from a genuine M59/66.

A factory-correct configuration is not a conversion. Rifles assembled by a state arsenal on inherited or reused receivers, such as a Finnish M27 built on an Imperial Russian receiver, are original as issued and need no conversion disclaimer. Describe the configuration plainly instead of disclaiming it.

Hedge provenance that has not been verified. Bring-back, capture, and unit-service claims stay "reported," "appears to be," "consistent with," or "not definitively established." Do not convert a family story or a single marking into a firm attribution: a Yokosuka property mark or a Vietnam-veteran bring-back Type 53 is described as reported, not established.

If a historical claim cannot be verified for the specific firearm, omit it or soften it on request.

Adverse research findings go to the operator, not into the listing. Aftermarket-versus-factory doubt, debunked provenance legends, bulk-import history, and price or collector-value commentary are reported in the chat reply so the operator can price and frame the item. They do not belong in `description.txt`. The listing handles them by declining to make the unsupported claim, not by arguing against it: a description is sales copy, not a research archive.

## Source Handling

Use the user's supplied facts as the primary source. Browse or otherwise verify external sources when adding model background, current references, or quoted/paraphrased information from named sites.

Preferred external background sources include reputable firearm publications, manufacturer pages, museum references, recognized collector references, or official historical documentation. Keep background directly relevant to the model or feature being described.

What the model paragraph may cover: who designed and built the model and where, including name changes of the maker; what it was built for and how it differs from its military or civilian sibling; factory variants and chamberings; production years; importers and import history; laws, executive orders and sanctions that affected its importation or sale, stated as history rather than advice; and how collectors classify it. What it may not cover: price or value commentary, rarity claims, and anything the Authenticity and Provenance section keeps out of the listing.

Cite only sources actually opened in this session, as a `Sources:` line at the end of the chat reply, never in the file. Never write a plausible-looking URL or reference from memory. If background is added without opening a source, say so rather than citing one.

A fact that appeared only in a search-result summary is not a source you opened. Either open the page that carries the fact or leave the fact out.

## The Title Line

Every description begins with a single `Title:` line. The importer takes the first `Title:` line as the per-gun WooCommerce product name (`Serial No.item_name`) and strips that line out of the customer-facing description, so it is never duplicated. A file with no `Title:` line makes the gun fall back to the shared model name on the store, and the importer's `resolve` step flags it `NO-TITLE`.

Write the title as a specific, front-loaded phrase of roughly 60 to 80 characters, built only from supplied facts. A workable order is year, manufacturer or arsenal, model, caliber/gauge, then one distinguishing fact such as matching status or condition.

```text
Title: 1943 Izhevsk M1891/59 Mosin Nagant 7.62x54R All-Matching Excellent
```

Keep the colon on the first line, plain text, no markdown, no emojis, no ALL CAPS.

## The California Answer Lines

The importer also reads two optional lines and writes them to the gun's POS record, where the store shows them as fields: `CA Legal:` (is this firearm legal to sell to a California buyer as configured?) and `Compliant Service:` (can the shop perform its paid California-compliance conversion on it?). Both are facts only the operator can supply, one about the law and one about the shop's own service. Never infer them from research or from the gun's features, and never write them unasked.

- Ask the operator for both answers once per gun, or once per batch when the operator gives a batch-wide rule, alongside the other research questions.
- Write each answered line at the top of the file, before the `Title:` line, exactly as `CA Legal: Yes`, `CA Legal: No`, `Compliant Service: Yes` or `Compliant Service: No`.
- An answer the operator did not give is left out entirely. Do not write a blank, "Unknown", "TBD" or a hedge: the importer treats any value other than Yes or No as ordinary prose, leaves the line in the customer-facing description and reports it, and a missing line correctly means "leave the gun's existing answer alone."
- Never mention California legality or the conversion service in the prose; the store renders the answers from the fields.

## Research Questions

After researching the firearm model, ask targeted questions about potentially notable features when they could materially improve the description, value context, or accuracy. Ask only questions that fit the specific firearm type and the user's supplied information.

Good question topics include matching numbers, serial number, receiver markings, arsenal or factory marks, import marks, crest or cartouche presence, stock markings, barrel length, choke, bore condition, sight variation, included accessories, sling, bayonet, case, magazine count, refinishing signs, date codes, proof marks, or model-specific variants.

Keep questions concise and practical. Do not ask a long generic checklist. Questions never delay the draft: the listing is written and saved first, and the `Questions to improve the listing` section follows it in the same reply. When the answers arrive, fold them into the saved file (Workflow step 5).

Carry batch-wide conventions forward instead of re-asking item by item. Once the user sets a rule for a lot, such as "standard caliber unless stated," apply it to every remaining item in that batch without asking again.

## Output Format

Use this structure by default:

1. The `CA Legal:` and `Compliant Service:` lines, each only when the operator answered it.
2. The `Title:` line.
3. A blank line.
4. One or two descriptive paragraphs written for shoppers and collectors.
5. A blank line.
6. The heading `Specifications`.
7. A clean key-value list with one specification per line.

Items 1 to 7 are the listing and are what goes into `description.txt`. The following belong in the chat reply only, never in the file, because the importer pushes the file to the product page as-is:

8. Optional word count only when the user asks for it or when helpful for confirming the minimum.
9. A `Sources:` line only when external sources were opened.
10. Optional `Questions to improve the listing` section when research or missing facts suggest useful follow-up questions, including the two California answers when the operator has not given them yet.

Keep labels plain and consistent. Use only fields supported by the input, choosing the ones that fit the gun. Typical labels, roughly in order:

```text
Specifications

Manufacturer:
Importer:
Country of origin:
Year:
Model:
Action:
Caliber:            (Gauge: for a shotgun)
Barrel length:
Configuration:
Serial:
Matching:
Condition:
Stock:              (Furniture: for polymer or modern furniture)
Finish:
Bore:
Rifling:
Magazines:
Muzzle:
Gas block:
Sights:
Features:
Included:
Mechanical condition:
```

Omit empty fields. Do not add a field just because it is common for the firearm type, and add a label not on this list when the gun calls for it. When a `Serial:` field is included, mask its last two digits.

## Style

Use direct, catalog-ready prose. Prefer clear phrases like "mechanically excellent," "bright bore with some pitting," "all-matching parts," or "minimal scratches and dings" when supplied by the user. Keep language polished but not exaggerated.

Do not use dramatic headlines, emojis, markdown tables, or overly long paragraphs unless the user requests them. Write plain text throughout: no `#` headings, `**bold**`, or bullet markers, because the text is pushed to a WooCommerce product description as-is.

One paragraph per line. Never break a paragraph across lines by hand: the store turns every newline into a line break, and the importer's merge of wrapped lines is a heuristic that treats a line ending in a period as the end of a paragraph. Separate paragraphs and the `Specifications` block with one blank line.

## Batch and Revision Handling

- "Same as" a prior lot or listing means reuse that listing's house style and swap in only the new facts. Do not redesign the format.
- "Wrong," "do it again," or "proper format" means discard the current draft entirely and rewrite in the requested template immediately. Never deliver a hybrid that keeps part of the rejected draft.
- Lot listings, such as a CZ85, Walther PP, M91/30, or M1957 bayonet lot, are representative rather than per-item. Describe the lot, note the variation across it, and say whether it is priced per unit or per set without giving a number. A lot has no single serial, so there is no serial folder to save under.

## Saving Description Files

Finish every description by saving it inside the firearm's serial-number folder under the working folder, whether the operator pointed at an existing batch folder or only gave the serial and the gun's facts in chat.

- Name the file exactly `description.txt`, lowercase. The importer prefers that name and otherwise falls back to the alphabetically first `.txt` in the folder, so a differently named file can be silently outranked by a stray text file.
- Write plain UTF-8 text with no markdown syntax. Save only the final listing text: the answered California lines, then the `Title:` line, the prose and the `Specifications` block. Research notes, the `Sources:` line, follow-up questions, and word counts stay in the chat reply, never in the file, even if the user asks to see them: they would be published on the product page.
- Use the serial number supplied by the user, found in the description, or visible as a folder name. Use the full, unmasked serial to locate and name the folder; the masked form appears only inside the listing text.
- Locate the serial folder under the working folder case-insensitively. If one exists, write into it and never rename it: folder names are what the importer's `--only` filter matches, case-sensitively, so a renamed folder means the gun is silently skipped on the next run.
- If no folder exists for the serial, create `<working folder>/<serial>/` and write the description there. The folder name is the serial exactly as the operator gave it, with no spaces, prefixes or masking. Before creating it, echo the serial back once in the reply (`Creating folder 5711517 under ~/Desktop/OSA batch 3 — correct?`) unless the operator already confirmed it or set a batch-wide rule; a folder named after a mistyped serial is a gun the importer reports as unknown and skips. The importer handles a folder without photos by setting the description and title only, so tell the operator the folder is waiting for photos (`main.*` is the primary photo).
- Report the folder name back to the user exactly as it appears on disk, preserving its case, so it can be passed to `--only` verbatim.
- Serial-number folders sit directly under the confirmed working folder (see Working Folder above), for example `~/Desktop/OSA website first batch/M915901979/description.txt`.
- If the serial folder contains an empty `description.txt`, write the final description into it.
- If `description.txt` already has content from an earlier batch, read it first and do not overwrite it unless the operator asked to update or replace it, or is supplying new facts for that gun. A file written or updated in the current batch is rewritten freely as answers and corrections come in.
- If no serial number can be identified from the request, the description or a folder name, ask for it before writing; a description without a serial has nowhere to live.
- After writing a batch, state how many files were written, how many folders were created, and confirm the file count equals the number of serial folders worked on.

## Handing Off to the Importer

Writing descriptions never starts an import and never asks whether to. After a batch, end the reply with the one-line hint `Import: /product-description-seo import` (plus the `--only` list when the run wrote only some of the folders). The import runs only when the operator invokes that command (see Commands). It invokes the `firearm-listing-import` skill and lets it run its own workflow and safety rails; do not reimplement them here. Pass it:

- `--root` = the working folder;
- `--only` = the folder names the operator listed, or, with none listed, the folders written in this session, exactly as they appear on disk, comma-separated; `import all` means every serial folder under the working folder and no `--only`;
- the note that a folder written without photos needs `attach --force` if the gun already has a gallery in the POS: the importer otherwise skips it as "already has gallery", and with no photos in the folder `--force` only rewrites the description, title and California answers, never the images;
- any `Photo note` raised in step 2, so the importer's own photo gate is not a surprise: it flags `MAIN-PORTRAIT` (portrait primary, refused with no override) and `PORTRAIT:<n>` (other portrait photos, refused unless `--allow-portrait`), and its step 1b has the agent look at every primary photo to confirm the gun is upright before anything is uploaded.

The importer's order is `resolve` (read-only preview, always first), its step 1b (look at every primary photo), `attach`, `push --channel woo`, `verify`, then its question about the New Arrivals email. Never ask for `--channel gunbroker` from here. If the `firearm-listing-import` skill is not available in the session, give the operator the three commands to run from that skill's folder instead of guessing at its internals:

```bash
uv run scripts/firearm_listings.py resolve --root "<working folder>" --only <A,B>
uv run scripts/firearm_listings.py attach  --root "<working folder>" --only <A,B>
uv run scripts/firearm_listings.py push    --root "<working folder>" --only <A,B> --channel woo
```

## Example

Input:

```text
1943 Izzy mfg receiver M1891/59 Mosin.
7.62x54
serial 5711517
two factory stamps
All matching
excellent condition, minimal scratches and dings in blonde wood stock and blued finish
complete with cleaning rod
strong rifling, bright bore with some pitting
mechanically excellent
```

The operator had set a batch rule earlier (`CA Legal Yes, Compliant Service No for the whole lot`), so both lines open the file; without an answer neither line would appear and both questions would sit under the draft. The full serial 5711517 is masked to 57115xx in the listing text while the folder is still named 5711517. The operator's facts make a 75-word first paragraph; the second paragraph came from an opened reference and was written without asking, as always. Everything below is produced in one reply.

Output pattern (the file):

```text
CA Legal: Yes
Compliant Service: No
Title: 1943 Izhevsk M1891/59 Mosin Nagant 7.62x54R All-Matching Excellent

This M1891/59 Mosin Nagant is built on a 1943 Izhevsk-manufactured receiver and is chambered in 7.62x54. It carries serial 57115xx, the receiver carries two factory stamps, and the rifle is all-matching. Overall condition is excellent, with only minimal scratches and dings in the blonde wood stock and the blued finish. The bore is bright with strong rifling and some pitting, and the rifle is mechanically excellent. It comes complete with its cleaning rod.

The M1891/59 is a Soviet carbine-length conversion of the M1891/30 rifle rather than a purpose-built carbine. The work cut the barrel and stock back to roughly the length of the M44 carbine, without the M44's side-folding bayonet, and the rear sight was regraduated for the shorter barrel, with the longer-range markings removed. Because the conversions reworked existing rifles, the receiver date and arsenal stamp record the original M1891/30 build, here Izhevsk in 1943, rather than the year the carbine was made; the programme itself is generally placed in the late 1950s, and the carbines are believed to have been held for reserve and training use before being released as surplus. Collectors of military surplus rifles catalog the 91/59 separately from the full-length 91/30 and from the M38 and M44, which left the factory as carbines.

Specifications

Manufacturer: Izhevsk
Country of origin: Soviet Union
Year: 1943
Model: M1891/59 Mosin Nagant
Caliber: 7.62x54
Serial: 57115xx
Matching: All matching
Condition: Excellent
Stock: Blonde wood stock with minimal scratches and dings
Finish: Blued finish with minimal scratches and dings
Bore: Bright bore with some pitting
Rifling: Strong
Features: Two factory stamps
Included: Cleaning rod
Mechanical condition: Mechanically excellent
```

Chat reply under the listing, not part of the file:

```text
Wrote ~/Desktop/<batch>/5711517/description.txt (1 of 1 folders, 0 created).
Sources: <URL of the reference actually opened>

Questions to improve the listing
- Is the bolt serial matching as well, or only receiver, butt plate and magazine floor plate?
- Any import mark on the barrel or receiver?

Import: /product-description-seo import
```

If the operator then replies `bolt matches too, no import mark`, the first paragraph and the `Matching:` line are rewritten in the saved file and the reply is one line: `Updated 5711517/description.txt`.
