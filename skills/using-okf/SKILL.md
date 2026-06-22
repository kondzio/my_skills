---
name: using-okf
description: Use when creating, updating, or authoring project knowledge documents in Open Knowledge Format (OKF) — markdown files with YAML frontmatter representing data assets, APIs, metrics, playbooks, or any concept in a knowledge bundle.
---

# Open Knowledge Format (OKF)

## Overview

OKF is a human- and agent-friendly format for knowledge: a directory of markdown files with YAML frontmatter. One file = one concept. The format is intentionally minimal — only `type` is required; everything else is optional or conventional.

**Version in use: OKF v0.1**

---

## Bundle Structure

```
bundle/
├── index.md                  # Optional. Progressive-disclosure listing.
├── log.md                    # Optional. Chronological update history.
├── <concept>.md              # A concept at the root.
└── <subdirectory>/
    ├── index.md
    ├── <concept>.md
    └── <subdirectory>/
        └── …
```

`index.md` and `log.md` are **reserved** — never use these names for concept documents.

---

## Concept Document Format

Every concept is a UTF-8 `.md` file with two parts:

### Frontmatter (YAML between `---` delimiters)

```yaml
---
type: <Type name>              # REQUIRED — identifies the kind of concept
title: <display name>          # Recommended
description: <one-line summary> # Recommended — used in index snippets and search
resource: <canonical URI>      # Recommended when concept maps to a real asset
tags: [tag1, tag2]             # Optional
timestamp: 2026-06-22T14:00:00Z # Optional — ISO 8601 last-modified
# any additional producer-defined keys are allowed
---
```

**`type` is the only required field.** Use descriptive, self-explanatory values — they are not registered centrally.

| Common `type` values | When to use |
|---|---|
| `BigQuery Table` | A BQ table concept |
| `BigQuery Dataset` | A BQ dataset |
| `API Endpoint` | A REST/gRPC endpoint |
| `Metric` | A business or technical metric |
| `Playbook` | An operational runbook or procedure |
| `Reference` | External documentation mirrored in the bundle |

### Body (markdown)

Free-form markdown. Favor structure (headings, tables, lists, code blocks) over prose — it aids both human reading and agent retrieval.

**Conventional section headings** (use when applicable):

| Heading | Purpose |
|---|---|
| `# Schema` | Columns/fields of a data asset |
| `# Examples` | Concrete usage examples, fenced code blocks |
| `# Citations` | Numbered external sources backing claims |
| `# Joins` | Cross-table relationships (common for tables) |

---

## Cross-Linking

Link concepts to each other using standard markdown links.

**Prefer absolute (bundle-relative) links** — stable when files are moved:
```markdown
See [customers table](/tables/customers.md) for the join key.
```

Relative links also work:
```markdown
See [neighboring concept](./other.md).
```

Consumers build a graph from these links. Broken links are tolerated — they may represent not-yet-written knowledge.

**Concept ID** = file path from bundle root minus `.md`. Example: `tables/users.md` → concept ID `tables/users`.

---

## Index Files (`index.md`)

No frontmatter. Body groups concepts under headings:

```markdown
# Tables

* [Orders](orders.md) - One row per completed customer order.
* [Customers](customers.md) - Registered customer accounts.

# Subdirectories

* [Playbooks](playbooks/) - Operational runbooks.
```

Entries should include the linked concept's `description` from frontmatter.

The bundle-root `index.md` MAY include a frontmatter block for declaring the OKF version:
```yaml
---
okf_version: "0.1"
---
```

---

## Log Files (`log.md`)

Optional but recommended — create one at any directory level where you want to track changes over time. Always create a root-level `log.md` when initializing a new bundle.

Chronological history, newest first. Date headings in `YYYY-MM-DD` form:

```markdown
# Directory Update Log

## 2026-06-22
* **Creation**: Added [orders table](/tables/orders.md).

## 2026-05-15
* **Initialization**: Created foundational directory structure.
```

Leading bold words (`**Update**`, `**Creation**`, `**Deprecation**`) are convention, not requirement.

---

## Citations

When body claims come from external sources, list them under `# Citations` at the bottom:

```markdown
# Citations

[1] [BigQuery schema reference](https://cloud.google.com/bigquery/docs/schemas)
[2] [Internal data quality runbook](https://wiki.internal/data/quality)
```

---

## Conformance Checklist

A bundle is conformant with OKF v0.1 when:

- [ ] Every non-reserved `.md` file has a parseable YAML frontmatter block
- [ ] Every frontmatter block has a non-empty `type` field
- [ ] `index.md` files follow the listing format (§6 of spec)
- [ ] `log.md` files use date-grouped entries, newest first (§7 of spec)

Consumers MUST NOT reject bundles with missing optional fields, unknown `type` values, unknown extra frontmatter keys, or broken cross-links.

---

## Quick Example

```markdown
---
type: BigQuery Table
title: Customer Orders
description: One row per completed customer order across all channels.
resource: https://console.cloud.google.com/bigquery?p=acme&d=sales&t=orders
tags: [sales, orders, revenue]
timestamp: 2026-06-22T14:00:00Z
---

# Schema

| Column        | Type      | Description                                      |
|---------------|-----------|--------------------------------------------------|
| `order_id`    | STRING    | Globally unique order identifier.                |
| `customer_id` | STRING    | FK into [customers](/tables/customers.md).       |
| `total_usd`   | NUMERIC   | Order total in US dollars.                       |
| `placed_at`   | TIMESTAMP | When the customer submitted the order.           |

# Joins

Joined with [customers](/tables/customers.md) on `customer_id`.

# Citations

[1] [BigQuery table schema](https://console.cloud.google.com/bigquery?p=acme&d=sales&t=orders)
```

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| Using `index.md` or `log.md` as concept filenames | Rename — those are reserved filenames |
| Missing `type` field | Add it — it's the only required frontmatter field |
| Relative links that break when files move | Use absolute bundle-relative links starting with `/` |
| Frontmatter in `index.md` body sections | Only the bundle-root `index.md` may have frontmatter (for `okf_version`) |
| Rejecting bundles with unknown type values | Consumers must tolerate unknown types gracefully |
| Putting log entries in oldest-first order | Log files go newest first |
