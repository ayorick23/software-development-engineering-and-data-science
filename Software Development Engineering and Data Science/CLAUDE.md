# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository type

This is **not a software codebase** — it is an **Obsidian vault** (personal knowledge base) of Markdown notes, version-controlled with git. There is no build, lint, or test tooling, no package manager, and no application code to run. "Working in this repo" means reading, writing, and cross-linking Markdown notes, not writing or executing programs.

Code snippets that appear inside notes (Python, SQL, Java, DAX, etc.) are teaching examples embedded in prose — they are not part of a runnable project and don't need their own tests or lint passes.

## Structure

Top-level folders group notes by subject:

- `Machine Learning/` — a numbered sequence of deep-dive notes (`01-...md` through `51-...md`) forming a continuous curriculum from ML fundamentals through MLOps, advanced Python, SQL, design patterns, and model deployment/monitoring. Notes explicitly build on prior ones (see cross-links below) — read the referenced earlier note before assuming context.
- `Python/`, `SQL/`, `Java/`, `R/`, `DAX/`, `Docker/`, `Git/`, `Power BI/`, `Tableau/`, `Calculus Fundamentals/`, `People Analytics/` — topical reference notes, one concept per file (e.g. `Python/Decorators.md`, `SQL/Joins.md`).
- `University Degree Notes/` — chronological class notes organized by `Ciclo 0N/<Materia>/Clase NN - <Título>.md`, in Spanish, mirroring an actual university course sequence.
- `Excalidraw/` — hand-drawn diagrams (`.excalidraw.md`) created via the Excalidraw plugin.
- `Templates/` — Templater plugin templates (e.g. `Clase.md`) used to scaffold new notes with frontmatter.
- `.obsidian/` — vault configuration and installed community plugins (Excalidraw, Templater, Calendar, Icon Folder, Outliner, TOC, Open in VS Code). Don't hand-edit plugin data files unless specifically asked to change vault configuration.

## Conventions to preserve when editing or adding notes

- **Language**: `Machine Learning/`, `University Degree Notes/`, and most prose explanations are in **Spanish**. `Python/`, `SQL/`, and similar quick-reference notes are in **English**. Match the existing language of the file/folder you're editing.
- **Wikilinks**: Notes cross-reference each other constantly using Obsidian wikilink syntax, including section anchors: `[[Note Name]]`, `[[Note Name#Section]]`, `[[Note Name#Section|display text]]`. When adding related content, link to existing notes instead of duplicating their explanations.
- **Frontmatter**: Many notes (especially in `Machine Learning/`) open with YAML frontmatter for `tags:` and `aliases:`. Class notes use the `Templates/Clase.md` frontmatter shape (`Fecha de creación`, `Materia`, `Fecha de clase`). Keep new notes consistent with the frontmatter pattern already used in their folder.
- **Numbered sequences**: Files in `Machine Learning/` and `Clase NN - ...` files within `University Degree Notes/` are strictly ordered and build on each other narratively (a note will say "Continúa de [[08-...]]"). When inserting a new note into one of these sequences, pick the next free number and add an explicit continuity link to the previous note, matching the existing style.
- **Structure within a note**: Long notes use `##`/`###` headers, `---` horizontal rules between major sections, bold key terms, and fenced code blocks for examples. Follow the surrounding note's section rhythm rather than introducing a new structure.
