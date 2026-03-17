# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

POC dashboard template — single `index.html`, no build step, no framework, no backend. Served as-is by GitHub Pages. All logic, data, and markup live in that one file.

## Running locally

```bash
python3 -m http.server   # or: npx serve .
```

Open `http://localhost:8000`. Can also be opened directly as a file in a browser.

---

## `index.html` structure

```
<head>
  Tailwind CDN + Lucide CDN + tailwind.config (darkMode:'class') + <style> overrides

<body>
  <!-- STATIC SHELL (never injected, always present) -->
  .flex.h-screen
  ├── <aside>  → sidebar logo + <nav id="sidebar-nav">   ← items injected by renderSidebar()
  └── .flex-1.flex-col
      ├── <header>                                        ← static, populated by renderHeader()
      │   ├── #btn-theme  / #icon-theme
      │   ├── #btn-notif  → #drop-notif [data-drop]
      │   │                  └── #notif-list             ← items injected by renderHeader()
      │   └── #btn-user   → #drop-user  [data-drop]
      │                      ├── #hdr-avatar, #drop-avatar
      │                      ├── #drop-name, #drop-email
      └── <main id="main">                               ← full page HTML injected here

  <!-- MODAL (always present, shown/hidden via .open) -->
  #modal-bg  →  #modal-title + #modal-body
```

### Static CSS rules (in `<style>`)

| Selector | Purpose |
|---|---|
| `[data-drop]` | Hidden by default (`display:none; position:absolute; z-index:50`) |
| `[data-drop].open` | Shown (`display:block`) |
| `.modal-bg` | Hidden by default |
| `.modal-bg.open` | Shown (`display:flex`) |
| `.line-through` | Text strikethrough for completed notes |
| `.group:hover .opacity-0` | Ensures hover-reveal works on dynamic content |

---

## JavaScript architecture

### State — global object `S`

```js
S.page           // active page id (string)
S.dark           // dark mode boolean

// Tasks page
S.period         // 'This Week' | 'Last Week' | 'This Month'
S.selectedDate   // number — day of month currently selected in schedule
S.weekStart      // number — first day of the visible 7-day week
S.tasks          // { 'This Week': [...], 'Last Week': [...], 'This Month': [...] }
S.schedule       // { [dayNumber]: [{ title, time, attendeeIds[] }] }
S.notes          // [{ id, title, description, completed }]

// People page
S.peopleQ        // search string
S.teamFilter     // team id or 'all'
S.viewMode       // 'list' | 'grid' | 'teams'
```

**Pattern:** mutate `S`, then call `renderPage()` (full re-render) or a targeted refresh like `refreshNotes()`.

### Function reference

| Function | What it does |
|---|---|
| `go(page)` | Navigate to a page — sets `S.page`, re-renders sidebar + main |
| `renderPage()` | Switches on `S.page`, writes HTML into `#main`, calls `icons()` |
| `renderSidebar()` | Rebuilds `#sidebar-nav` from `NAV[]`, calls `icons()` |
| `renderHeader()` | Populates avatar/name/email/notifications in the static header |
| `placeholder(title, body)` | Returns HTML for a simple content card (used by stub pages) |
| `tasksHTML()` | Returns full HTML for the Tasks page |
| `afterTasks()` | Binds `#period-btn` click after Tasks page renders |
| `peopleHTML()` | Returns full HTML for the People page |
| `afterPeople()` | Binds `#people-q` input + `#team-btn` click after People renders |
| `personCard(person, compact)` | Returns HTML for a single person card |
| `notesHTML()` | Returns HTML for the notes list (used by both initial render and refresh) |
| `refreshNotes()` | Updates only `#notes-list` — avoids full page re-render |
| `toggleNote(id, v)` | Toggle note completed state |
| `delNote(id)` | Delete a note |
| `openAddNote()` / `saveNewNote()` | Modal flow for adding a note |
| `openEditNote(id)` / `saveEdit(id)` | Modal flow for editing a note |
| `setPeriod(p)` | Set task time filter and re-render |
| `selDate(d)` | Select a schedule day and re-render |
| `navWeek(dir)` | `dir = 1` or `-1`, advances/retreats `S.weekStart` by 7 |
| `setAssignee(taskId, personId)` | Reassign a task and re-render |
| `setTeam(id)` | Set people team filter and re-render |
| `setView(m)` | Set people view mode and re-render |
| `modal(title, bodyHtml)` | Open modal with given title and body HTML |
| `closeModal()` | Close modal |
| `toggleDrop(e, id)` | Toggle a `[data-drop]` element open/closed |
| `closeDrops()` | Close all `[data-drop]` elements |
| `toggleTheme()` | Toggle dark mode — flips `S.dark`, toggles `dark` class on `<html>` |
| `av(person, size)` | Returns avatar `<img>` HTML with initials fallback |
| `icons()` | Calls `lucide.createIcons()` — must be called after any `innerHTML` write |
| `getPerson(id)` | Find person by id, falls back to `people[0]` |
| `getTeam(id)` | Find team by id |
| `isOnline(person)` | Returns true if current time is within person's working hours |

### Data arrays

**`people[]`** — 16 entries. Shape:
```js
{ id:'people_N', name, imageURL, email, workingHours:{start,end,timezone}, team }
```
Current user: `people_11` (Lucy Pearl).

**`teams[]`** — 5 entries. Shape:
```js
{ id, name, color }  // color is a Tailwind class string for badges
```
Team ids: `dev`, `qa`, `design`, `product`, `marketing`.

**`S.tasks[period][]`** — Shape:
```js
{ id, name, comments, likes, assigneeId, status, sc }
// sc = Tailwind badge classes e.g. 'bg-green-100 text-green-800'
```
Status values: `'In Progress'` (green), `'Pending'` (pink), `'Completed'` (blue).

**`S.schedule[dayNumber][]`** — Shape:
```js
{ title, time, attendeeIds[] }
```

**`S.notes[]`** — Shape:
```js
{ id, title, description, completed }
```

### Sidebar nav — `NAV[]`

```js
const NAV = [
  { id:'tasks',     label:'My Tasks',  icon:'check-square' },
  { id:'dashboard', label:'Section 1', icon:'bar-chart-3' },
  { id:'projects',  label:'Section 2', icon:'folder' },
  { id:'documents', label:'Section 3', icon:'file-text' },
  { id:'receipts',  label:'Section 4', icon:'receipt' },
  { id:'people',    label:'People',    icon:'users' },
];
```

Icons are Lucide icon names. Add/rename entries here to change the sidebar.

---

## How to add a new page

1. Add an entry to `NAV[]`:
   ```js
   { id:'reports', label:'Reports', icon:'chart-bar' }
   ```

2. Add a `case` in `renderPage()`:
   ```js
   case 'reports': main.innerHTML = reportsHTML(); afterReports(); break;
   ```

3. Write the render function:
   ```js
   function reportsHTML() {
     return `<div class="p-6">...</div>`;
   }
   function afterReports() {
     // bind any events that need getElementById after render
   }
   ```

4. Add any new state fields to `S` if needed.

5. Call `icons()` at the end of `afterReports()` if the render function doesn't go through `renderPage()` (which already calls `icons()`).

---

## UI patterns

### Card

```html
<div class="bg-white dark:bg-gray-800 rounded-lg border border-gray-200 dark:border-gray-700 p-6">
  <h3 class="font-semibold flex items-center mb-4">
    <i data-lucide="icon-name" class="w-5 h-5 mr-2"></i>Card Title
  </h3>
  <!-- content -->
</div>
```

### Button (outline)

```html
<button class="flex items-center px-3 py-1.5 text-sm border border-gray-300 dark:border-gray-600 text-gray-700 dark:text-gray-300 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-700">
  <i data-lucide="plus" class="w-4 h-4 mr-2"></i>Label
</button>
```

### Button (primary / filled)

```html
<button class="px-4 py-2 text-sm bg-gray-900 dark:bg-white text-white dark:text-gray-900 rounded-lg hover:bg-gray-700 dark:hover:bg-gray-100">
  Label
</button>
```

### Badge

```html
<span class="px-2 py-0.5 rounded-full text-xs font-medium bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-300">
  Label
</span>
```

### Text input / Textarea

```html
<input type="text" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-lg text-sm bg-white dark:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-blue-500">
<textarea class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-lg text-sm bg-white dark:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-blue-500 resize-none"></textarea>
```

### Dropdown (inline, inside a `relative` wrapper)

```html
<div class="relative">
  <button onclick="toggleDrop(event,'drop-myid')" class="...">Label</button>
  <div id="drop-myid" data-drop class="right-0 top-10 bg-white dark:bg-gray-800 border border-gray-200 dark:border-gray-700 rounded-lg shadow-lg py-1 min-w-40">
    <button onclick="myAction(); closeDrops();" class="block w-full text-left px-4 py-2 text-sm hover:bg-gray-100 dark:hover:bg-gray-700">Option</button>
  </div>
</div>
```

`data-drop` elements need to live inside a `position:relative` parent so the absolute positioning works.
Dropdowns that are bound via `addEventListener` after render must call `e.stopPropagation()` to prevent the document-level `closeDrops()` listener from immediately closing them.

### Avatar with fallback (`av()` helper)

```js
av(person, 'w-8 h-8')  // returns <img> + fallback <span> with initials
```

Wrap in `<div class="flex items-center">` when shown next to text.

### Modal

```js
modal('Modal Title', `<div class="space-y-4">...form fields...</div>`);
// close with:
closeModal()
```

Read form values from inside `#modal-body` using `document.getElementById('m-title')` etc.

### Icon

```html
<i data-lucide="icon-name" class="w-4 h-4"></i>
```

Always call `icons()` after writing icons via `innerHTML`. Browse icon names at lucide.dev.

### Empty state

```html
<div class="text-center py-12 text-gray-500 dark:text-gray-400">
  <i data-lucide="inbox" class="w-12 h-12 mx-auto mb-4 opacity-50"></i>
  <h3 class="text-lg font-medium mb-1">Nothing here</h3>
  <p class="text-sm">Descriptive message</p>
</div>
```

---

## GitHub Pages deployment

Enable Pages in repo settings → source: **Deploy from branch** → branch `main`, folder `/` (root). No build process needed — `index.html` is served directly.
