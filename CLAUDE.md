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
- `aws/` — AWS / Cloud Concepts
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