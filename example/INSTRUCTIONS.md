# INSTRUCTIONS.md — POC Generator

This file tells you exactly what to build and how. Read it completely before touching any code.

---

## What you must produce

A single file: **`index.html`** at the workspace root (not inside `example/`).

This file will be deployed to GitHub Pages and opened directly in a browser. No server, no build step, no compilation. Everything — markup, styles, and logic — lives in that one file.

---

## Starting point

Use `example/index.html` as your base. It is a fully working single-page application with the correct shell structure, dark mode, sidebar, header, modal, dropdowns, and JavaScript patterns.

Read `example/CLAUDE.md` — it documents every function, pattern, and UI component in the template. Follow those conventions exactly.

**Do not use any other framework or structure.** Copy the template's approach for state (`S`), routing (`go(page)` / `renderPage()`), event binding (`afterXxx()`), icons (`icons()` after every `innerHTML` write), and UI patterns (cards, badges, buttons, modals, dropdowns).

---

## What to customize

Replace every element of the template with content relevant to the requirement. Nothing should remain generic.

### App identity
- `<title>` — use the product or system name from the requirement
- Sidebar logo / `h1` — same as above

### Sidebar navigation — `NAV[]`
Replace the generic sections (`Section 1`, `Section 2`, etc.) with the actual pages the system needs. Every entry in `NAV[]` must have a fully implemented page — **no `placeholder()` pages in the final output**.

### Data
Replace the generic `people[]`, `teams[]`, `tasks`, `schedule`, `notes`, `notifications` with data that makes sense for the domain:
- Use realistic names, emails, roles, statuses, and values
- Make the data internally consistent (e.g. assignees reference real people, categories match the domain)
- Provide enough records to make the UI look populated and credible (aim for 8–16+ entries in lists, 3–5+ per category)

### Pages
Implement every page listed in `NAV[]` with real content and full interactivity:
- CRUD flows where relevant (create, edit, delete via modal)
- Filters, search, or sorting where it makes sense for the domain
- Charts or metrics using inline SVG or a CDN library (e.g. Chart.js) where useful
- Tables, cards, and detail views consistent with the template's visual style

---

## Technical constraints

- **Pure HTML + CSS + vanilla JavaScript only.** No React, Vue, Angular, Svelte, TypeScript, or any tool requiring a build step.
- **CDN libraries are allowed** via `<script src="https://cdn...">`. Examples: Chart.js, Alpine.js, Flatpickr, etc.
- **No absolute URL paths.** The file is served from a GitHub Pages subdirectory. `/dashboard` will 404. Use hash-based navigation (`#section`) or JS show/hide — the template already handles this via `go(page)`.
- **No external API calls** that require authentication or backend services. All data is in-memory.
- **Dark mode must work.** The template uses Tailwind's `dark:` classes and the `dark` class on `<html>`. Keep all new UI elements dark-mode compatible.

---

## Quality bar

This will be demoed to a real client. It must look and feel like a finished product.

- **No `lorem ipsum`**, no placeholder text, no `TODO` comments, no "Coming soon" sections.
- **Every visible UI element must work.** Buttons must do something. Forms must submit. Filters must filter.
- **Realistic mock data.** Numbers, dates, names, and values should feel plausible for the domain.
- **Consistent visual design.** Follow the card, badge, button, input, and dropdown patterns from `example/CLAUDE.md`. Do not invent new design patterns.
- **Scope ruthlessly.** If you can't implement something fully, remove it entirely. A smaller app that works perfectly is better than a larger app with broken flows.

---

## Checklist before finishing

- [ ] `index.html` exists at the workspace root
- [ ] App title and sidebar logo reflect the domain
- [ ] Every `NAV[]` entry has a fully implemented page (no `placeholder()` calls)
- [ ] All placeholder data replaced with domain-relevant content
- [ ] Dark mode works on every new page and component
- [ ] No hardcoded absolute paths (`/anything`) — only hash routing or JS navigation
- [ ] `icons()` called after every `innerHTML` write
- [ ] No broken buttons, empty modals, or unresponsive filters
- [ ] No `lorem ipsum`, `TODO`, or placeholder text anywhere
