# 2026 Expense Planner

A single-file HTML expense tracker with persistent storage, interactive timeline, and cash flow visualization. Built iteratively in a Claude chat session; designed to be maintained in a Claude Code project going forward.

---

## Architecture

**Single file:** `expense-planner.html` (or `expense-planner-standalone.html` for local use)

All HTML, CSS, and JS live in one file. No build step, no dependencies beyond CDN-loaded Chart.js and Google Fonts.

**One file:**
- `expense-planner.html` — uses `localStorage` (works in any browser, open directly)

---

## Data Layer

Expenses are stored as a JSON array. Two sources of truth:

### 1. `DEFAULT_EXPENSES` (in-file seed data)
At the top of the `<script>` block. Used when storage is empty or after a Reset. This is what you update when maintaining the file via Claude Code.

```js
const DEFAULT_EXPENSES = [
  {
    id: 1,                        // integer (legacy) or uid string e.g. 'e3x9z2m'
    name: 'Some Purchase',
    amt: 250,                     // number, dollars
    month: 5,                     // 1-12
    priority: 'committed',        // 'committed' | 'optional' | 'nicetohave'
    purchased: true,              // optional boolean, omit if false
    note: 'Optional context'      // optional string
  },
  // ...
];
```

### 2. `localStorage` (runtime state)
Key: `expenses_2026`
Value: `JSON.stringify(expenses)`

Runtime edits (status changes, month moves, price edits, new items) are persisted here with a 1-second debounce. On load, storage takes precedence over `DEFAULT_EXPENSES`.

### Feeding data from an export
Use the **⬆ Export** button in the UI to copy current state as JSON. To update the file baseline:
1. Export JSON from running instance
2. Paste into Claude Code session
3. Claude replaces the `DEFAULT_EXPENSES` array
4. Commit updated file
5. On next load, hit **↻ Sync** to merge any new defaults into existing storage, or **Reset** (triple-click) to reload from defaults entirely

---

## Storage API (standalone version)

```js
// Save
localStorage.setItem('expenses_2026', JSON.stringify(expenses));

// Load
const stored = localStorage.getItem('expenses_2026');
expenses = stored ? JSON.parse(stored) : [...DEFAULT_EXPENSES];

// Delete
localStorage.removeItem('expenses_2026');
```

**Debounce:** All saves are debounced 1000ms to avoid write storms on rapid UI interaction.

---

## IDs

New items get a pseudorandom uid: `'e' + Math.random().toString(36).slice(2, 9)`

Always starts with `'e'` to avoid numeric-literal parse errors in inline onclick handlers. Legacy DEFAULT_EXPENSES items use integer ids — the `findById` helper uses loose equality (`==`) to handle both:

```js
const findById = id => expenses.find(e => e.id == id);
```

---

## Key Functions

| Function | Description |
|---|---|
| `initData()` | Load from storage or fall back to defaults. Call once on page load. |
| `save()` | Debounced write to storage. Call after any mutation. |
| `render()` | Re-renders list, timeline, KPIs, and chart. |
| `renderList()` | Expense item rows with inline controls. |
| `renderTimeline()` | Monthly breakdown with planned vs. spent-so-far. |
| `renderKPIs()` | Top summary cards including leftover savings accumulator. |
| `renderChart()` | Running cash balance line chart. |
| `addExpense()` | Reads add-form inputs, pushes new item, saves, re-renders. |
| `deleteExpense(id)` | Filters out by id, saves, re-renders. |
| `cycleStatus(id)` | Cycles committed → optional → nicetohave → purchased → committed. |
| `updateMonth(id, val)` | Updates month, saves, re-renders. |
| `updateAmt(id, val)` | Updates amount, saves, re-renders KPIs/timeline/chart only. |
| `editName(id)` | Replaces name div with inline input; commits on blur/Enter. |
| `exportData()` | Copies current `expenses` JSON to clipboard. |
| `syncDefaults()` | Merges new DEFAULT_EXPENSES entries (by id) into storage. |
| `handleReset()` | Triple-click confirmation with depleting timer bar + boom animation. |
| `initForm()` | Populates month/year dropdowns dynamically from current date. |

---

## Constants

```js
const NET_MO   = 1422.58;   // Monthly discretionary cash after all fixed expenses & savings
const MONTHS   = ['Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'];
const MONTH_NO = [3,4,5,6,7,8,9,10,11,12];
const ANNUAL   = NET_MO * 10; // Mar–Dec = 10 months
```

**NET_MO** is the key figure — update this if monthly net income or fixed expenses change.

---

## UI Controls (per item)

| Control | What it does |
|---|---|
| Emoji status button | Cycles priority/purchased state |
| Month `<select>` | Moves item to a different month |
| Amount `<input>` | Inline price edit, commits on blur |
| ✏️ pencil button | Inline name edit, commits on Enter/blur |
| ✕ button | Deletes item |

---

## Header Buttons

| Button | What it does |
|---|---|
| ⬆ Export | Copies current JSON to clipboard (paste to Claude for sync) |
| ↻ Sync | Merges new DEFAULT_EXPENSES entries not in storage |
| ⚠ Reset | Triple-click + 3s timer to clear storage and reload defaults |

---

## Timeline Logic

- **Past months** (before current month): dimmed, labeled "past / was leftover"
- **Current month**: highlighted with ← now marker
- **Per-month display:** planned total, spent-so-far (purchased items only), leftover bar
- **Leftover savings KPI:** accumulates surplus from all completed months

---

## Suggested Claude Code Workflow

```
expense-planner/
├── expense-planner.html   # Open directly in any browser
├── CLAUDE.md              # Claude Code project context
└── README.md              # This file
```

1. Open `expense-planner.html` locally for day-to-day use
2. When data needs updating: Export JSON → paste to Claude Code → Claude updates `DEFAULT_EXPENSES` → commit
3. When adding features: work in Claude Code, test locally, commit
4. Use Export button as the bridge between local state and source of truth in the file

---

## Known Limitations

- `window.storage` (Claude artifact version) is rate-limited — rapid interactions can trigger `Internal server error`. Debounce mitigates this but doesn't eliminate it entirely.
- `window.storage` does not work on Claude iOS mobile — use standalone version locally or make edits on desktop.
- MONTHS/MONTH_NO currently hardcoded to Mar–Dec 2026. For a 2027 planner, update these constants and the `ANNUAL` calculation.
- No multi-year support yet — all items assumed to be in 2026 unless year dropdown is added to the timeline logic.
