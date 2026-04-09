# PRD: GTD Mind Sweep Tool

Version: 1.2
Status: Ready for Claude Code

---

## Overview

A self-contained HTML tool that guides the user through a comprehensive GTD mind sweep across personal and professional trigger categories. Captures items per category, flags each as a Task or Project, then exports a structured summary for Claude to process via OmniFocus MCP.

---

## Architecture

**Phase 1 (this tool):** HTML/CSS/JS — no dependencies, no build step, runs in any browser from a local file.

**Phase 2 (separate):** User pastes export into Claude (Productivity & Project Management project). Claude parses the JSON block and creates OmniFocus inbox tasks or project placeholders via MCP.

**Critical requirement:** The trigger list must be defined as a single, separate data structure at the top of the file — completely decoupled from UI rendering logic. The UI renderer iterates over `Object.keys(TRIGGER_LIST)` to build sections dynamically, so adding a third top-level section (or more) requires zero UI code changes.

---

## Session Persistence

All captured items, Task/Project flags, and expand/collapse state are saved to `localStorage` on every change (debounced ~300ms). On load, the tool restores the previous session automatically. A "Clear session" button in the header wipes localStorage after a confirmation prompt. Storage key: `gtd-mind-sweep-v1`.

---

## Data Structure

The trigger list is defined as a single JavaScript constant at the top of the file. The UI renderer iterates over `Object.keys(TRIGGER_LIST)` to build sections dynamically.

```javascript
const TRIGGER_LIST = {
  "Personal": {
    "Projects & Commitments": [
      "Projects started, not completed",
      "Projects that need to be started",
      "Commitments/promises to others (spouse, partner, children, parents, family, friends, professionals)",
      "Returnable items"
    ],
    "Communications": [
      "Calls to make or return",
      "Emails to send or respond to",
      "Cards, letters, thank-you's to write",
      "Texts to follow up on"
    ],
    "Upcoming Events": [
      "Birthdays, anniversaries, weddings, graduations",
      "Holidays, travel, vacations",
      "Dinners, parties, receptions",
      "Cultural events, sporting events",
      "School events, parent-teacher conferences"
    ],
    "Family": [
      "Projects/activities with spouse or partner",
      "Projects/activities with children",
      "Projects/activities with parents or relatives",
      "Conversations I need to have"
    ],
    "Administration": [
      "Bills to pay",
      "Filing, paperwork, forms",
      "Home office supplies and equipment",
      "Data backup, cloud storage",
      "Subscriptions to review or cancel"
    ],
    "Home/Household": [
      "Repairs needed (plumbing, electrical, HVAC, roof)",
      "Remodeling or construction projects",
      "Landscaping, driveways, garage",
      "Furniture, decor, walls, floors, ceilings",
      "Appliances, lights, fixtures",
      "Kitchen, laundry, cleaning, organizing",
      "Storage areas to clear or reorganize"
    ],
    "Technology (Personal)": [
      "Computers — hardware, software, updates",
      "Phones — setup, repairs, upgrades, settings",
      "Smart home devices",
      "Streaming services, media libraries",
      "Password manager, security, accounts",
      "Photos — backup, organization, albums",
      "TV and entertainment setup",
      "Internet/network issues"
    ],
    "Financial": [
      "Bills, payments, autopay review",
      "Bank accounts, statements",
      "Investments, retirement accounts",
      "Loans, mortgage",
      "Taxes — preparation, estimated payments, filings",
      "Budget review, spending tracking",
      "Insurance — review coverage, claims",
      "Legal affairs — wills, trusts, estate planning"
    ],
    "Health": [
      "Doctor, dentist, specialist appointments",
      "Checkups due or overdue",
      "Prescriptions to refill",
      "Diet, food, nutrition adjustments",
      "Exercise routines, equipment, memberships",
      "Mental health, counseling",
      "Sleep, recovery"
    ],
    "Personal Development": [
      "Classes, seminars, certifications",
      "Books to read",
      "Courses — online or in-person",
      "Creative projects or expressions",
      "Career-adjacent learning",
      "Faith and spiritual growth"
    ],
    "Leisure": [
      "Music, podcasts, videos to explore",
      "Travel — places to visit, trips to plan",
      "People to visit or connect with",
      "Hobbies, sports equipment, supplies",
      "Cooking, recipes to try",
      "Photography"
    ],
    "Transportation": [
      "Vehicle maintenance, repairs",
      "Registration, insurance, inspections",
      "Commuting changes",
      "Bikes, recreational vehicles"
    ],
    "Clothes & Personal Items": [
      "Professional wardrobe gaps",
      "Casual, formal, athletic wear",
      "Accessories, luggage",
      "Repairs, tailoring, donations"
    ],
    "Errands": [
      "Pharmacy, grocery, hardware store",
      "Bank, post office",
      "Gifts to buy",
      "Returns or exchanges"
    ],
    "Community & Service": [
      "Neighborhood, neighbors",
      "Civic involvement, local government",
      "Volunteer commitments",
      "Schools, parent involvement"
    ],
    "Faith & Church": [
      "Bible study — passages, resources, plans",
      "Family discipleship — conversations, activities, rhythms",
      "Elder responsibilities — meetings, pastoral care, decisions pending",
      "Safety team — training, protocols, equipment, volunteers, incidents to review",
      "Service commitments — upcoming or recurring",
      "Hospitality — people to have over, follow-through from past gatherings"
    ],
    "Waiting For (Personal)": [
      "Mail orders, deliveries",
      "Items out for repair",
      "Reimbursements owed",
      "Loaned items to get back",
      "RSVPs awaited",
      "Information or decisions from others"
    ]
  },
  "Professional": {
    "Projects & Commitments": [
      "Projects started, not completed",
      "Projects that need to be started",
      "'Look into' projects",
      "Commitments/promises to boss, partners, colleagues",
      "Commitments to subordinates or direct reports",
      "Commitments to customers or external organizations",
      "Decisions that need to be made — who needs to know?"
    ],
    "Communications": [
      "Calls to make or return",
      "Emails to send, respond to, or follow up on",
      "Voicemails, memos, formal letters",
      "Slack or Teams messages to respond to",
      "Conversations I've been avoiding"
    ],
    "Writing & Documents": [
      "Reports, evaluations, reviews to finish",
      "Proposals to write or submit",
      "Articles, summaries, instructions",
      "Status updates, meeting minutes",
      "Rewrites and edits in progress"
    ],
    "Meetings": [
      "Upcoming meetings to prepare for",
      "Meetings that need to be scheduled",
      "Meetings that need debrief or follow-up action"
    ],
    "Read/Review": [
      "Industry articles, reports, research",
      "Books, periodicals",
      "Websites, newsletters, email subscriptions",
      "Documents awaiting my review or approval"
    ],
    "Financial (Professional)": [
      "Budget reviews, forecasts",
      "P&L, balance sheet",
      "Receivables, payables",
      "Expense reports to submit",
      "Vendor contracts or invoices",
      "Credit line, banking matters"
    ],
    "Planning & Strategy": [
      "Goals, targets, objectives to review or set",
      "Business plans, marketing plans",
      "Upcoming events, conferences, travel to plan",
      "Presentations to prepare"
    ],
    "Organization & Operations": [
      "Org chart, reporting structure",
      "Job descriptions, role clarity",
      "Change initiatives in progress",
      "Culture, morale, team health"
    ],
    "Administration": [
      "Legal issues, compliance",
      "Insurance, risk management",
      "Policies and procedures to update",
      "Training programs or requirements"
    ],
    "Staff & Team": [
      "Hiring, onboarding",
      "Performance reviews, feedback due",
      "Staff development conversations",
      "Compensation, recognition",
      "Delegation — things I've handed off but not confirmed complete"
    ],
    "Systems & Tools": [
      "Software — licenses, updates, migrations",
      "Hardware, office equipment",
      "Databases, file systems, storage",
      "AI tools and workflows to set up or review",
      "Automations that may have broken",
      "Business cards, stationery, supplies"
    ],
    "Sales & Marketing": [
      "Customer relationships to tend",
      "Prospects, leads, pipeline",
      "Proposals outstanding",
      "Marketing materials to update",
      "Public relations items"
    ],
    "Professional Development": [
      "Training, certifications, seminars",
      "Skills to develop or practice",
      "Things to learn or research",
      "Books, courses, podcasts",
      "Career planning, resume, positioning",
      "Networking — people to connect with"
    ],
    "Waiting For (Professional)": [
      "Delegated tasks not yet confirmed complete",
      "Replies to communications",
      "Responses to proposals",
      "Decisions by others that are blocking progress",
      "Items submitted for approval or reimbursement",
      "External dependencies (implementations, deliveries, etc.)"
    ]
  }
};
```

---

## UI Structure & Behavior

### Layout

- Clean, minimal, print-friendly design
- Top-level collapsible sections rendered dynamically from `TRIGGER_LIST` keys (currently Personal and Professional)
- Each section contains collapsible category cards
- All sections and categories collapsed by default
- "Expand All" / "Collapse All" toggle at the top of each section

### Category Card

Each category card contains:

1. **Header row** — category name + chevron toggle + item count badge. Badge shows **total captured items** in the category, regardless of flag state.
2. **Trigger prompts** — bulleted read-only list of triggers (visible when expanded), serving as memory joggers only
3. **"I have items here" checkbox** — checking this opens the capture area
4. **Capture area** (hidden until checkbox checked):
   - A list of capture rows, each containing:
     - A text input field (full width, expands with content)
     - A Task/Project toggle button (default: Task)
     - A remove row button (×)
   - An "Add item" button to append a new row
   - Pressing Return in a **non-empty** text field adds a new row and focuses it
   - Pressing Return in an **empty** text field does nothing (prevents runaway empty rows)
   - Task/Project toggle is a semantic `<button>` — Tab to focus, Space or Enter to flip state
   - Placeholder text on first field: *"Describe the task or project…"*
5. **Visual indicator** on card header when items have been captured (subtle highlight + count badge)

### Task/Project Toggle

- Each captured line item has an inline toggle: **[ Task ] [ Project ]**
- Default state: Task
- Toggle is a simple two-state button — no modal, no extra fields
- Visual distinction: Task = neutral, Project = accented color

### Navigation & Progress

- Sticky header with:
  - Tool title: "GTD Mind Sweep"
  - Progress indicator: *"14 of 38 categories reviewed"* (reviewed = opened at least once)
  - **Clear session** button (wipes localStorage after confirmation)
  - **Export button** — always visible, active when at least one non-empty item has been captured across all categories. Empty rows are trimmed and excluded from export. If a category has only empty rows, it does not appear in the export.
- No forced linear progression — user can open any category in any order

---

## Export Format

Triggered by the Export button. Produces **two formats** side by side in the modal: a human-readable plaintext view (for sanity-checking) and a JSON block (for Claude to parse). The JSON is the source of truth for Phase 2.

**Human-readable view (top of modal, read-only):**

```
GTD MIND SWEEP EXPORT
Generated: 2026-04-08 19:42
---

[Personal > Projects & Commitments]
- [TASK] Call accountant about estimated taxes
- [PROJECT] Plan kitchen remodel

[Professional > Staff & Team]
- [TASK] Schedule quarterly review with Jennifer

---
Total items: 3 (2 tasks, 1 project)
```

**JSON block (below the plaintext, in a code fence — this is what gets copied):**

```json
{
  "version": "1.0",
  "generated": "2026-04-08T19:42:00-04:00",
  "totals": { "items": 3, "tasks": 2, "projects": 1 },
  "items": [
    { "section": "Personal", "category": "Projects & Commitments", "type": "task", "text": "Call accountant about estimated taxes" },
    { "section": "Personal", "category": "Projects & Commitments", "type": "project", "text": "Plan kitchen remodel" },
    { "section": "Professional", "category": "Staff & Team", "type": "task", "text": "Schedule quarterly review with Jennifer" }
  ]
}
```

**Rules:**

- Only categories with at least one non-empty item appear in the export
- Empty rows are trimmed before export
- Full `[Section > Category]` path used as header for every block in the plaintext view
- Summary count at bottom (total, tasks, projects)

**Export UI:**

- Clicking Export opens a modal containing the human-readable preamble and the JSON block
- **Copy button** copies the JSON block only (not the human-readable preamble)
- **Copy plaintext** button copies the human-readable version instead
- Instruction text in modal: *"Paste this into Claude in the Productivity & Project Management project."*
- Close button to dismiss modal

**Modal accessibility:**

- Esc closes the modal
- Focus is trapped within the modal while open
- On close, focus returns to the Export button
- Modal uses `role="dialog"` and `aria-modal="true"`

**Filename convention (for future download feature):** `mind-sweep-YYYY-MM-DD.json`

---

## Claude Import Instructions

When the user pastes a mind sweep export, Claude should:

1. Parse the JSON block (ignore the human-readable preamble if present)
2. **Sanitize smart quotes and apostrophes** (`'` → `'`, `"` `"` → `"`) before passing text to OmniFocus MCP — batch operations can fail on smart quotes
3. For each item with `"type": "task"`: create one OmniFocus inbox task, name = `text` verbatim (post-sanitization)
4. For each item with `"type": "project"`: create one OmniFocus project placeholder in the inbox, name = `text` verbatim, with a default next action of "Define next action"
5. If batch creation fails, fall back to sequential creation (`createSequentially: true`)
6. No due dates, tags, or project assignment beyond the above
7. Confirm on completion: *"Created X inbox tasks and Y project placeholders from your mind sweep."*

---

## Out of Scope (v1)

- Editing or reordering items after export
- Direct Claude/MCP integration from within the tool
- Mobile optimization
- Print stylesheet
- Drag-to-reorder items within a category

---

## File Delivery

- Single `.html` file
- No external dependencies (no CDN, no frameworks)
- Vanilla HTML/CSS/JS only
- Runs from local filesystem (`file://`) without a server
- Filename: `gtd-mind-sweep.html`

---

## Deployment

**Primary use case:** Open `gtd-mind-sweep.html` as a local file on Mac (`file://`).

**iPad / cross-device use case:** iOS Safari does not reliably open local `.html` files from the Files app. To use the tool on iPad, host it on **GitHub Pages**:

1. Create a public GitHub repo (e.g., `gtd-mind-sweep`)
2. Commit `gtd-mind-sweep.html` as `index.html` at the repo root
3. Enable GitHub Pages in repo Settings → Pages → Source: `main` branch, root folder
4. Bookmark the resulting URL (`https://<username>.github.io/gtd-mind-sweep/`) on iPad Safari

**localStorage behavior across deployments:**

- localStorage is scoped per-origin, so the hosted version and the local-file version each have their own independent session — sweeps do not sync between them
- On iOS Safari, localStorage is "best effort" and can be cleared under storage pressure or by aggressive tracking-prevention settings. For occasional use this is unlikely to cause data loss, but it is less bulletproof than macOS Safari or Chrome
- If losing a session mid-sweep on iPad becomes a problem, fall back to Mac for long sweeps and use iPad for shorter ones

**No code changes required** to support GitHub Pages — the single HTML file works identically whether opened locally or served over HTTPS.

---

## Success Criteria

- User can complete a full sweep in a single browser session
- Session survives accidental tab close or browser crash via localStorage
- Every captured item is flagged as Task or Project before export
- Export produces clean, unambiguous output Claude can process without clarifying questions
- Trigger list can be updated by editing only the `TRIGGER_LIST` data structure — zero UI code changes required, including adding new top-level sections
- Tool works offline, in any modern browser, opened as a local file

---

## Intro Line for Claude Code Session

> "Build the tool described in this PRD. Deliver a single `gtd-mind-sweep.html` file. No external dependencies. Vanilla HTML/CSS/JS only. Use Plan Mode — show me the plan before writing code."
