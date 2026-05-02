# Living Model

**Author:** Chris Selland & Nik Bear Brown
**Publisher:** Bear Brown, LLC
**Status:** Draft
**Started:** 2026-05-02

## Structure

```
chapters/
    00-frontmatter.md   ← copyright, dedication, preface
    01-introduction.md  ← Chapter 0 / Introduction
    02-chapter-01.md    ← Chapter 1
    ...
    04-chapter-03.md    ← Chapter 3
    99-back-matter.md   ← acknowledgments, about the author, notes, references, index
```

## Chapters

| File | Title | Status |
|------|-------|--------|
| 00-frontmatter.md | Front Matter (copyright, dedication, preface) | ☐ |
| 01-introduction.md | Introduction | ☐ |
| 02-chapter-01.md | Chapter 1 | ☐ |
| 03-chapter-02.md | Chapter 2 | ☐ |
| 04-chapter-03.md | Chapter 3 | ☐ |
| 99-back-matter.md | Back Matter (acknowledgments, notes, references, index) | ☐ |

## Build

```bash
./build.sh
```

Output goes to `output/` (gitignored).

## Figures

```bash
./graphs.sh
```

Processes `<!-- → [TYPE: description] -->` comments in every chapter:
- Tabular figures → classed markdown tables (`.infographic-table`, `.comparison-table`, `.data-table`)
- Non-tabular figures → placeholder images in `images/`, ready to replace
- CSS log appended to `styles/kindle-book.css` on each run

Review `chapters/*-updated.md`, then promote:
```bash
for f in chapters/*-updated.md; do mv "$f" "${f/-updated/}"; done
```

## Styles

| File | Purpose |
|------|---------|
| `styles/kindle.css` | Shared base — typography, figure table classes. Do not edit per book. |
| `styles/kindle-book.css` | Book-specific overrides. Edit freely. `graphs.sh` appends its log here. |

## Publish

Upload `output/living-model.epub` to [KDP](https://kdp.amazon.com).
