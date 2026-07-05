# CLAUDE.md — Endocrine Imaging Trainer

Guidance for Claude Code when working in this repository.

## What this is

A small, self-contained set of **case-based imaging trainers** for an endocrinology
fellowship team. Not a production app — **keep the architecture tiny**: no build
system, no framework, no backend. Each trainer is **one standalone HTML file** with
embedded CSS + JS + a JSON `IMAGES` data array. The only external dependency is the
Google Fonts "Inter" stylesheet. Data (progress, annotations, comments) lives in the
browser's `localStorage`. Deployed via **GitHub Pages** from the repo root.

## Layout

```
index.html               ← landing hub; links to each standalone trainer
pit_trainer.html         ← Pituitary MRI module (85 cases, 255 MCQs)
adrenal_trainer.html     ← Adrenal CT/MRI module (128 cases, 402 MCQs, 66 descriptive)
endocrine_trainer.html   ← older Tailwind hub prototype (not the deployed entry)
pit_image_data/          ← pituitary images (batch 1)
pit_image2/              ← pituitary images (batch 2)
adrenal_image1/          ← adrenal images used by adrenal_trainer.html (128 .jpg)
adrenal_image2/          ← images staged for the NEXT adrenal module (not yet built)
.nojekyll .github/       ← GitHub Pages static deploy (do not remove .nojekyll)
```

When adding a module, the trainer HTML lives at the **repo root** and references
images by **relative path** (e.g. `adrenal_image1/20260705....jpg`) so paths resolve
both locally and on Pages.

## Data schema (the `IMAGES` array)

Each case is one object. Two mutually exclusive question types:

```jsonc
{
  "id": 1,                                  // sequential, assigned at merge time
  "file": "adrenal_image1/2026....jpg",     // relative path from the HTML
  "figNum": "Fig 3 (Trauma hematoma)",      // source figure label, optional
  "category": "Hemorrhage",                 // drives the gallery filter + tag
  "tagClass": "tag-hemorrhage",             // CSS class for the coloured tag
  "title": "Post-Traumatic Adrenal Hematoma",
  "caption": "…expert caption…",            // HIDDEN until the user reveals it

  // EITHER an MCQ case:
  "mcqs": [
    { "q": "stem?", "opts": ["a","b","c","d"], "ans": 1, "exp": "why", "hint": "nudge" }
  ]

  // OR a descriptive case (used when the source image has NO caption):
  // "descriptive": [ { "q": "describe…?", "model": "model answer" } ]
}
```

- `ans` is the **0-indexed** correct option. Every MCQ has exactly 4 options.
- A case has **`mcqs` OR `descriptive`**, never both.
- Categories currently in use: Normal, Adenoma, Pheochromocytoma, Carcinoma,
  Metastasis, Myelolipoma, Cyst, Hemorrhage, Hyperplasia, Other, Technique,
  Characterization. Add a matching `tag-*` CSS rule if you introduce a new one.

## Question authoring rules (agreed with the user)

1. **3 MCQs** per single-image case.
2. **5 MCQs** when one image file is a multi-panel figure (many sub-images / rich
   figure, e.g. a washout series or an algorithm chart).
3. If the image has **no caption**, write **3 descriptive questions** (open-ended,
   **no options**) with a model answer instead of MCQs.
4. Base questions on **facts**. When a caption is present, **derive the MCQs from the
   caption** — the numbers, the diagnosis, the teaching point.
5. In the app the **caption is hidden while questions are asked**; the user reveals it
   deliberately. Never restate the whole caption inside a stem.

## The three features that differ from the pituitary reference

`adrenal_trainer.html` mirrors `pit_trainer.html`'s theme but adds:

1. **Dynamic question count** — the UI reads `img.mcqs.length` (not a hardcoded 3).
2. **Save-able annotation + comment tool** — a `<canvas>` overlay for freehand drawing
   (colour swatches, undo, clear) plus a comment `<textarea>`, both persisted per case
   in `localStorage` (key `adrenal_trainer_v1`).
3. **Descriptive question mode** — renders a stem with a "Reveal model answer" button
   and no options, for uncaptioned images.

## Repeatable process for a NEW imaging folder

Use the **`add-imaging-module`** skill (`.claude/skills/add-imaging-module/`). In short:

1. Read every image in the new folder in **small batches**, and **immediately** author
   each batch's case objects into a numbered JSON file in a scratch `batches/` dir.
   (Authoring incrementally survives context limits — don't hold 100+ cases in memory.)
2. Apply the authoring rules above (3 / 5 / descriptive).
3. Merge all `batch_*.json` into one array, assign sequential `id`s, validate JSON.
4. Copy `adrenal_trainer.html` as the template for the new module, swap in the new
   `IMAGES` array and accent colour, keep the annotation/descriptive machinery.
5. Add a module card to `index.html`.
6. Verify: JSON parses, embedded `<script>` passes `node --check`, image paths resolve.
7. Do **not** commit or push unless the user asks. Images must be committed for Pages
   to serve them.

## Conventions

- Keep everything in the dark theme defined by the `:root` CSS variables; give each
  module its own accent (`--accent`) — pituitary = blue, adrenal = amber/orange.
- No external JS/CSS beyond the Inter font. Everything inline.
- `.nojekyll` must stay so Pages serves files/underscored folders verbatim.
