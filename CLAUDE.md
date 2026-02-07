# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **documentation-only repository** containing educational lessons about Microsoft Entra ID (formerly Azure Active Directory). There is no application code, build system, or tests.

## Content Structure

The repository contains 10 lessons organized as follows:

```
lesson-XX-topic/
├── README.md              # Main lesson content (or overview if sub-lessons exist)
├── X.1-subtopic/
│   ├── README.md          # Sub-lesson content
│   └── diagram.drawio     # Visual diagram for the sub-lesson
├── X.2-subtopic/
│   └── ...
```

**Restructuring Status**: Lesson 01 has been restructured into sub-lessons. Lessons 02-10 need the same treatment:
- Split each lesson into logical sub-lessons (numbered X.1, X.2, etc.)
- Each sub-lesson folder contains a README.md and diagram.drawio file
- Update main lesson README.md to be an overview linking to sub-lessons

## Diagram Format

Diagrams use **Draw.io XML format** (`.drawio` files):
- Can be opened with diagrams.net (web), VS Code Draw.io extension, or desktop app
- Each sub-lesson should have an accompanying diagram illustrating key concepts
- Use consistent styling: rounded boxes, color coding by category, clear labels

## Writing Guidelines

When creating or modifying lessons:
- Use tables for comparisons and feature lists
- Include practical examples (PowerShell, KQL queries, YAML configurations)
- Structure each sub-lesson with: Overview, Learning Objectives, Content sections, Diagram reference, Key Takeaways, Next navigation
- Link between related lessons and sub-lessons
- Reference official Microsoft documentation URLs
