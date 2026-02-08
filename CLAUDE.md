# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **documentation-only repository** containing educational lessons about Microsoft Entra ID (formerly Azure Active Directory). There is no application code, build system, or tests.

## Content Structure

The repository contains 10 lessons organized as follows:

```
lesson-XX-topic/
├── README.md              # Lesson overview linking to sub-lessons
├── X.1-subtopic/
│   ├── README.md          # Sub-lesson content
│   ├── diagram.drawio     # Visual diagram source (Draw.io XML)
│   └── diagram.png        # Exported diagram image
├── X.2-subtopic/
│   └── ...
```

All 10 lessons have been restructured into sub-lessons with consistent formatting.

## Diagram Format

Diagrams use **Draw.io XML format** (`.drawio` files):
- Can be opened with diagrams.net (web), VS Code Draw.io extension, or desktop app
- Each sub-lesson has an accompanying diagram with exported PNG image
- Styling conventions:
  - Container/group boxes: sharp corners (`rounded=0`)
  - Content boxes: rounded corners (`rounded=1`)
  - Color coding by category
  - Clear labels and consistent sizing

## Writing Guidelines

When creating or modifying lessons:
- Use tables for comparisons and feature lists
- Include practical examples (PowerShell, KQL queries, YAML configurations)
- Structure each sub-lesson with: Overview, Learning Objectives, Content sections, Diagram reference, Key Takeaways, Next navigation
- Link between related lessons and sub-lessons
- Reference official Microsoft documentation URLs
