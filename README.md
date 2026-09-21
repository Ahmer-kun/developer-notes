# Developer Notes

Personal technical notes, references, and things worth remembering while learning and building software.

## Contents

- [INDEX.md](INDEX.md): every note, grouped by topic
- [ROADMAP.md](ROADMAP.md): what's written, what's in progress, what's planned
- [CHANGELOG.md](CHANGELOG.md): what changed and when

## Why this exists

Notes scattered across notebooks, bookmarks, and old chat windows are hard to find again. This repository is the single place for them: short explanations, commands, comparisons, and troubleshooting checklists worth being able to find again. It doubles as a learning log.

The notes aim to be practical rather than complete. If something is worth writing down, it's usually because it's confusing, easy to forget, or shows up while debugging.

## Topics

| Area | What's in it |
|---|---|
| [programming](programming/) | JavaScript fundamentals: closures, event loop, promises, references vs values |
| [backend](backend/) | HTTP, CORS, cookies and sessions, JWT, authentication vs authorization |
| [computer-science](computer-science/) | TCP vs UDP, DNS, database indexes and transactions |
| [cybersecurity](cybersecurity/) | hashing vs encryption |
| [devops](devops/) | Git (undo, rebase, recovery, cheatsheet), Linux CLI |
| [debugging](debugging/) | stack traces, EADDRINUSE, env variables, CORS errors |

More areas are listed in the [roadmap](ROADMAP.md). Folders are only created when there's a note to put in them.

## How I use it

Notes get added while studying, building projects, debugging problems, and revisiting concepts I only half understood. Existing notes get corrected or expanded when I find something wrong or missing, so older notes aren't frozen.

Some notes are short reference sheets. Others go deeper on a concept. Comparison notes (`x-vs-y.md`) exist for pairs of things that are frequently confused. Debugging notes follow a fixed shape: symptoms, likely causes, checks, fix, why it happened, prevention.

## A caveat

These are notes, not authoritative documentation. I try to verify technical claims against official docs, RFCs, or the project's own documentation, and I list references where it's useful. Version-dependent details are marked as such. If something is wrong, an issue or pull request is welcome.

## Status

Actively maintained.
