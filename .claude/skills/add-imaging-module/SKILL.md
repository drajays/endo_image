---
name: add-imaging-module
description: Build a new endocrine imaging MCQ/descriptive trainer module from a folder of case images, mirroring the pituitary/adrenal trainer architecture. Use when the user adds a new imaging folder (e.g. adrenal_image2, thyroid_image1) and wants a standalone HTML trainer with hidden captions, board-style questions, and the save-able annotation tool.
---

# Add an Imaging Trainer Module

Build one self-contained HTML trainer from a folder of case images, matching the
existing modules (`pit_trainer.html`, `adrenal_trainer.html`). Read `CLAUDE.md` at the
repo root first — it holds the data schema, question rules, and conventions this skill
depends on.

## Guardrails

- **Tiny architecture only.** One standalone HTML file: embedded CSS + JS + a JSON
  `IMAGES` array. No build tools, no framework, no backend. Only external dep is the
  Inter Google Font. Persist user data in `localStorage`.
- **Never commit or push unless the user explicitly asks.**
- Keep the dark theme; give the new module its own `--accent` colour.

## Step 1 — Author cases incrementally (survives context limits)

Do **not** try to read all images then author all at once — you will lose work to
context truncation. Instead, loop in small batches:

1. Create a scratch dir: `<scratchpad>/batches/`.
2. Read ~4–6 images at a time with the Read tool (it shows the image + any caption).
3. **Immediately** write those cases as `batch_NN.json` (a JSON array of case objects)
   before reading the next batch.
4. Repeat until every image in the folder is authored. Track which files are done with:
   `grep -ho '<timestamp>[0-9]*\.jpg' batches/batch_*.json | sort -u | wc -l`

### Authoring rules (from CLAUDE.md)
- Single image → **3 MCQs**. Multi-panel figure → **5 MCQs**.
- Image with **no caption** → **3 descriptive questions** (`descriptive: [{q, model}]`,
  no options).
- Derive MCQs from **facts**, preferring the caption's numbers/diagnosis/teaching point.
- Each MCQ: `{ q, opts:[4], ans:<0-indexed>, exp, hint }`.
- Give each case: `file` (relative path), `figNum`, `category`, `tagClass`, `title`,
  `caption`. Reuse existing category names / `tag-*` classes where possible; add a CSS
  rule for any new tag.

## Step 2 — Merge & validate

```python
import json, glob
cases=[]
for f in sorted(glob.glob('batches/batch_*.json')): cases.extend(json.load(open(f)))
for i,c in enumerate(cases,1): c['id']=i
json.dump(cases, open('module_data.json','w'), ensure_ascii=False, indent=1)
```

Confirm: distinct image count == folder file count == number of case objects, and every
batch parses (`python3 -c "import json;json.load(open(f))"`).

## Step 3 — Assemble the HTML

Use `adrenal_trainer.html` as the template (it already has the dynamic question count,
the `<canvas>` annotation tool + comment box with `localStorage`, and the descriptive
renderer). Copy it, then:

- Replace the `IMAGES` array with the merged data (inject via a script that swaps a
  `/*IMAGES_PLACEHOLDER*/` token so you never hand-paste 100+ objects).
- Change the `<title>`, header text, `--accent`/`--accent2`, and the `LS_KEY`
  (e.g. `thyroid_trainer_v1`) so modules don't share saved state.
- Keep `CAT_NAMES` in sync with the categories you used.

Place the file at the **repo root**; image `file` paths are relative to it.

## Step 4 — Wire into the hub

Add a `<a class="card live …">` module card to `index.html` with the module's case /
MCQ / descriptive counts and a link to the new HTML file.

## Step 5 — Verify

- Embedded script passes a syntax check: extract the last `<script>` block and run
  `node --check`.
- The `/*IMAGES_PLACEHOLDER*/` token is gone and the case count is embedded.
- Open the file locally (or spot-check a few `file` paths exist on disk).

Remind the user that new image files must be committed for GitHub Pages to serve them,
but only commit/push when they ask.
