# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

We're building the app described in `docs/specs/`

Keep replies concise and focused on key information. No unnecessary fluff, no long code snippets.

## Project status

This repository (`apogee`) has no code yet. It holds a single commit with a placeholder `README.md`. There is no language, build system, test framework or architecture to document.

## Specs

Features are specified in `docs/specs/` before they are built:
- `constitution.md` holds the project-wide principles, constraints and tech stack. Every feature must follow it.
- Each feature has its own folder with `requirements.md` (user stories and acceptance criteria), `design.md` (architecture, interfaces and data models) and `tasks.md` (a checklist to work through in order, ticking items off as they are done).
- `feature-name/` is the template. Copy it for each new feature.

Update this file once the project is scaffolded. Add:
- Build, lint and test commands, including how to run a single test
- The high-level architecture: the main components and how they interact
