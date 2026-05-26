# expense-planner — Claude Code Context

## What this is
A single-file HTML expense tracker for 2026. No build step, no dependencies except CDN-loaded Chart.js and Google Fonts. All HTML, CSS, and JS live in `expense-planner.html`.

## The one file that matters
**`expense-planner.html`** — open this directly in any browser. Uses `localStorage` key `expenses_2026`.

## Data workflow
The file has two sources of truth:

1. **`DEFAULT_EXPENSES`** — seed array at the top of the `<script>` block. This is the canonical source in the repo. Update this when the user pastes in exported JSON.
2. **`localStorage`** — runtime state. Takes precedence on page load.

**Typical update flow:**
1. User hits ⬆ Export in the browser → copies current JSON to clipboard
2. User pastes JSON here into Claude Code
3. Claude replaces the `DEFAULT_EXPENSES` array in `expense-planner.html`
4. Commit the updated file
5. User hits ↻ Sync in the browser to merge new defaults into existing storage

## Key constants (update when finances change)
```js
const NET_MO   = 1422.58;   // Monthly discretionary cash after fixed expenses & savings
const MONTHS   = ['Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'];
const MONTH_NO = [3,4,5,6,7,8,9,10,11,12];
const ANNUAL   = NET_MO * 10; // Mar–Dec = 10 months
```

## Expense object shape
```js
{
  id: 1,                   // integer (legacy) or uid string 'eXXXXXXX'
  name: 'Description',
  amt: 250,                // dollars, number
  month: 5,                // 1–12
  priority: 'committed',   // 'committed' | 'optional' | 'nicetohave'
  purchased: true,         // optional boolean — omit if false
  note: 'Context'          // optional string
}
```

## IDs
- Legacy items: integer
- New items: `'e' + Math.random().toString(36).slice(2, 9)` — always starts with `'e'`
- `findById` uses loose equality (`==`) to handle both

## Editing rules
- Never touch the debounce logic or the reset animation without being asked
- When replacing `DEFAULT_EXPENSES`, preserve the exact object shape and field order
- Don't add a build system, bundler, or npm dependencies — intentionally zero-toolchain
