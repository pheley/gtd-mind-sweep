---
name: mind-sweep-import
description: >
  Processes GTD Mind Sweep export JSON and creates OmniFocus tasks and projects
  via the OmniFocus MCP. Use this skill whenever the user says "process my mind
  sweep", "import mind sweep", "mind sweep import", or pastes JSON that contains
  a "version" field, a "totals" object, and an "items" array where each item has
  "section", "category", "type" (task or project), and "text" fields. Also
  trigger when the user pastes text starting with "GTD MIND SWEEP EXPORT" (the
  plaintext variant) so we can redirect them to copy the JSON instead.
---

# Mind Sweep Import

You are processing a GTD Mind Sweep export. The user ran a mind sweep in an HTML
tool that walks through personal and professional trigger categories, captured
action items, flagged each as Task or Project, and exported structured JSON.
Your job is to get those items into OmniFocus quickly and accurately.

## Step 1: Detect format

Look at what the user pasted:

- **If it starts with `GTD MIND SWEEP EXPORT`** or is clearly plaintext (no JSON
  structure), respond with:
  > That looks like the plaintext export. Go back to the Mind Sweep tool and
  > click **Copy JSON** instead.
  Then stop. Do not attempt to parse plaintext into items.

- **If it contains a JSON object** (possibly with a plaintext preamble above it),
  extract the JSON block and continue to Step 2.

## Step 2: Parse and validate

Parse the JSON. It should have this shape:

```json
{
  "version": "1.0",
  "generated": "...",
  "totals": { "items": N, "tasks": N, "projects": N },
  "items": [
    { "section": "...", "category": "...", "type": "task"|"project", "text": "..." }
  ]
}
```

If the JSON is malformed or missing the `items` array, tell the user what went
wrong and ask them to re-export.

## Step 3: Sanitize text

macOS and iOS sometimes auto-substitute curly quotes. OmniFocus MCP batch
operations can choke on them, so sanitize every item's `text` field before
sending it to OmniFocus:

| Find | Replace with |
|------|-------------|
| `\u2018` (left single quote) | `'` (straight apostrophe) |
| `\u2019` (right single quote) | `'` (straight apostrophe) |
| `\u201C` (left double quote) | `"` (straight double quote) |
| `\u201D` (right double quote) | `"` (straight double quote) |

Do not alter the text in any other way. Item names go to OmniFocus verbatim
after this sanitization pass.

## Step 4: Build the batch

Construct a single `batch_add_items` call. For each item in the `items` array:

- `"type": "task"` items become `{ "type": "task", "name": "<sanitized text>" }`
- `"type": "project"` items become `{ "type": "project", "name": "<sanitized text>" }`

Do not add due dates, defer dates, tags, flags, folder assignments, estimated
minutes, or any other fields. Tasks land in the OmniFocus inbox. Projects are
created as empty project shells at the root level (zero actions inside them).

## Step 5: Send to OmniFocus

Call `batch_add_items` with the full items array.

If the call **fails**, retry once with `createSequentially: true`. If it still
fails, report the error to the user.

## Step 6: Confirm

On success, respond with:

> Created **X** inbox tasks and **Y** projects from your mind sweep.

Use the actual counts from what was successfully created. Keep the confirmation
short — the user doesn't need a list of every item.
