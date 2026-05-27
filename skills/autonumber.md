---
name: autonumber-designer
description: >
  Design autonumber (sequential numbering) configurations for database-based applications and output a ready-to-use JSON spec with a live preview of generated values.

  Use this skill whenever the user wants to design, configure, or define an autonumber scheme — including invoice numbers, order IDs, receipt numbers, purchase order numbers, employee codes, asset tags, ticket numbers, or any identifier that combines a prefix, suffix, and auto-incrementing number. Also trigger for phrases like "sequential numbering", "document numbering", "auto-increment with prefix", "number series", "running number", "serial number format", or "reset numbering every year/month". If the user mentions any kind of structured ID that increments automatically, use this skill — even if they don't say "autonumber".
---

# Autonumber Designer

Help the user design an autonumber configuration for a database application. Your output is a clean JSON spec plus a preview of what the generated numbers will look like.

## What to gather

Collect these fields from the user (confirm with sensible defaults if not all are provided):

| Field | Description | Example |
|-------|-------------|---------|
| `name` | A label for this sequence | "Invoice Number" |
| `prefix` | Text placed before the number | `"INV-"`, `"ORD/"`, `""` |
| `suffix` | Text placed after the number | `"-SG"`, `""` |
| `leading_zeroes` | Total digit width of the number part (zero-padded) | `4` → `0001` |
| `start_number` | First number in the sequence | `1` |
| `step_number` | Increment between consecutive numbers | `1` (or `5`, `10`, etc.) |
| `stop_number` | Maximum number before reset or overflow | `9999` |
| `reset_criteria` | When/how the counter resets | See below |

### Reset criteria types

Support all of these:

| Type | Meaning |
|------|---------|
| `never` | Sequence grows indefinitely; `stop_number` acts as a warning threshold only |
| `on_stop` | When `stop_number` is reached, cycle back to `start_number` |
| `yearly` | Reset to `start_number` at the start of each calendar year |
| `monthly` | Reset to `start_number` at the start of each calendar month |
| `daily` | Reset to `start_number` at the start of each day |
| `field_based` | Reset to `start_number` whenever a specified field value changes (e.g., per customer, per branch, per document type) |

**For time-based resets** (`yearly`, `monthly`, `daily`), the time part is usually embedded in the prefix or suffix using a placeholder. Suggest this to the user if they haven't already — e.g., prefix `"INV-{YYYY}-"` for yearly resets so each year's numbers start fresh and are visually distinct.

**For `field_based`**, ask what field drives the reset (e.g., `customer_id`, `branch_code`).

## Clarification strategy

If the user gives a natural-language description (e.g. "invoice numbers that reset every year"), fill in sensible defaults and confirm them before generating. A quick confirmation round is better than outputting something wrong and iterating.

Good defaults for business document numbers:
- leading_zeroes: `4`
- start_number: `1`
- step_number: `1`
- stop_number: `9999`
- reset_criteria: `yearly` (with `{YYYY}` in prefix)

## Placeholder conventions

Use these in `prefix` and `suffix` values when time or context needs to be embedded:

| Placeholder | Meaning |
|-------------|---------|
| `{YYYY}` | 4-digit year, e.g. `2024` |
| `{YY}` | 2-digit year, e.g. `24` |
| `{MM}` | 2-digit month, e.g. `01`–`12` |
| `{DD}` | 2-digit day, e.g. `01`–`31` |
| `{FIELD}` | The value of the field driving a `field_based` reset |

In `format_pattern` (shown in the preview), also use `{NNNN}` to represent the number slot (use as many N's as `leading_zeroes`).

## JSON output format

Always produce output in this structure:

```json
{
  "autonumber": {
    "name": "Invoice Number",
    "prefix": "INV-{YYYY}-",
    "suffix": "",
    "leading_zeroes": 4,
    "start_number": 1,
    "step_number": 1,
    "stop_number": 9999,
    "reset_criteria": {
      "type": "yearly",
      "reset_to": 1
    }
  },
  "preview": {
    "format_pattern": "INV-{YYYY}-{NNNN}",
    "first_values": [
      "INV-2024-0001",
      "INV-2024-0002",
      "INV-2024-0003"
    ],
    "approaching_stop": [
      "INV-2024-9997",
      "INV-2024-9998",
      "INV-2024-9999"
    ],
    "after_reset": [
      "INV-2025-0001",
      "INV-2025-0002"
    ],
    "capacity_note": "9,999 unique values per year (step: 1)"
  },
  "notes": [
    "The {YYYY} placeholder in the prefix is replaced by the current 4-digit year at generation time.",
    "After reaching INV-{YYYY}-9999, the counter resets to 1 on January 1 of the next year."
  ]
}
```

### Preview rules

- `first_values`: First 3 generated numbers in the sequence.
- `approaching_stop`: Last 3 numbers before hitting `stop_number`. If `stop_number` ≤ 3, show only what's available.
- `after_reset`: First 2 numbers after a reset. For `never` type, omit this field and set `approaching_stop` to a note like `"... grows past 9999"`.
- For `step_number > 1`, space values accordingly: e.g. step 5 → `0001, 0006, 0011`.
- For `field_based`, use a concrete example in preview values: e.g. `"CUST001-0001"` if the field is a customer code.
- Use today's actual year for `{YYYY}` in preview values.

### Capacity note format

| Reset type | Note format |
|------------|-------------|
| `never` | `"Unlimited — grows past {stop_number}"` |
| `on_stop` | `"{N} unique values per cycle (step: {step})"` |
| `yearly` | `"{N} unique values per year (step: {step})"` |
| `monthly` | `"{N} unique values per month (step: {step})"` |
| `daily` | `"{N} unique values per day (step: {step})"` |
| `field_based` | `"{N} unique values per {field} value (step: {step})"` |

Where `N = floor((stop_number - start_number) / step_number) + 1`.

## Validation checks

Flag these problems before generating (don't silently proceed):

- **Overflow risk**: `stop_number` has more digits than `leading_zeroes` allows — e.g., stop=10000 with leading_zeroes=4 means the last value `10000` breaks the format. Suggest increasing `leading_zeroes` to 5.
- **Invalid range**: `start_number >= stop_number` — this is an error.
- **Step too large**: `step_number > (stop_number - start_number)` — the sequence would produce only one value before stopping.
- **Time-based without date placeholder**: If `reset_criteria.type` is `yearly`, `monthly`, or `daily` but neither prefix nor suffix contains a date placeholder, warn that values from different periods will look identical and suggest adding one.

## Multiple sequences

If the user needs more than one sequence (e.g. invoices + purchase orders + receipts), wrap everything in an array:

```json
{
  "sequences": [
    { "autonumber": { ... }, "preview": { ... }, "notes": [ ... ] },
    { "autonumber": { ... }, "preview": { ... }, "notes": [ ... ] }
  ]
}
```

## Tone

Most users of this skill are developers or business analysts who know what they want. Be direct: confirm the config in one message, then output. If something looks unusual (e.g., step of 100 on an invoice number), mention it briefly but don't belabour it. The JSON output should be clean, copy-paste ready, and accompanied by a short plain-English summary of what the autonumber does.
