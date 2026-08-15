# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repository is a **single-source-of-truth AI Engineering interview preparation handbook**. It is a content-only knowledge base — there is no source code, no build system, no test suite, and no linter to run. Files are Markdown notes organized into topic folders.

## Repository Structure

The repo is flat by design: one folder per interview topic, each containing its own `README.md` as the entry point. Subtopic notes live as additional Markdown files alongside the topic README. No `.gitkeep` placeholders should be removed — they reserve subfolder structure that may be filled in later.

Topics (defined in the root `README.md`):
- `dsa/` — DSA & Complexity
- `system-design/` — System Design
- `ml-dl-nlp/` — ML, DL & NLP
- `cloud/` — Cloud (AWS primary, Azure as cross-reference)
- `genai-agenticai/` — GenAI & Agentic AI
- `project-experience/` — Previous Company Project Experience

## Topic Format (Authoring Convention)

Every topic note should ideally contain these six sections, per the root `README.md`:
1. Concept
2. Simple explanation
3. Key interview points
4. Common interview questions
5. Practical examples
6. Quick revision notes

When adding a new note, place it inside the relevant topic folder and follow this structure so notes remain interchangeable across topics.

## Working Conventions

- This is **not** a code repository — do not introduce package managers, test runners, or build tooling.
- Keep edits in Markdown. Prefer relative links between topic notes rather than absolute paths.
- The root `README.md` is the table of contents and the canonical source for the topic list and topic format. Update it when adding or renaming topics.
- Topic-level `README.md` files are placeholders (`Add interview preparation notes here.`) until real content is added per the topic format above.

## Interview Brush-Up Doc Naming

Interview brush-up notes follow a uniform filename pattern so they are easy to find and sort:

`<Topic Name> interview brushup guide.md`

Rules:
- Use **spaces**, not underscores or hyphens, between words.
- The **topic name is Title Case** (e.g. `Tree Based Classifier`, `MCP`, `RAG`).
- The suffix `interview brushup guide` is **lowercase** and identical for every file.
- Place the file directly in the relevant topic folder.
- Reference example: `ml-dl-nlp/Tree Based Classifier interview brushup guide.md`.

When adding a new brush-up doc, follow this pattern. When renaming or refactoring old ones (e.g. `MCP_Interview_Brush_Up_Guide.md`, `RAG_Interview_Brush_Up_Guide.md`), convert them to the same pattern.