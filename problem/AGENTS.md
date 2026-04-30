# AGENTS.md

## Project: Colorful Conversations

Colorful Conversations is a public listening and problem-mapping initiative. The goal is to collect real problems people are facing, organize them into themes, and display the conclusions publicly through a Kumu map.

Public Kumu map:

https://kumu.io/srmrox/colorful-conversations#problems

The public map should show synthesized themes, relationships, causes, consequences, tensions, and possible solutions. It must not expose raw private responses, usernames, handles, phone numbers, screenshots, private messages, or source links unless explicitly approved.

---

## Core product principle

Separate the system into two layers:

1. **Private raw feedback layer**
   - Stores original responses from Reddit, LinkedIn, Instagram, WhatsApp, and other channels.
   - May include raw text, source platform, source type, timestamp, handle, source URL, notes, consent status, and suggested tags.
   - Must remain private by default.

2. **Public insight layer**
   - Stores only approved public themes, nodes, connections, summaries, counts, and anonymized notes.
   - Feeds the Kumu graph.
   - Must never include private identifiers or unapproved raw responses.

Kumu is the public visualization layer. The database or files maintained by this project are the source of truth.

---

## Current public graph structure

Kumu expects a JSON-style graph blueprint with two main arrays:

```json
{
  "elements": [],
  "connections": []
}
```

Each `element` is a node in the map.

Each `connection` is an edge between two nodes.

Suggested node types:

- Theme
- Cause
- Consequence
- Concept
- Skill
- Solution
- Gap
- Affected Group
- Open Question

Suggested clusters:

- Civic Behaviour and Responsibility
- Future Readiness and Skills
- Digital Society, Privacy, and Trust

---

## Current themes and concepts

The initial issues identified so far are:

1. Rights Without Responsibilities
   - People are aware of rights but not responsibilities.
   - This may lead to self-centred behaviour, weak civic responsibility, and reduced social trust.

2. Lack of Future-Ready Skills
   - People may not have the skills needed to adapt to the future.
   - Related areas include adaptability, digital literacy, AI readiness, reskilling, and employment insecurity.

3. Loss of Privacy in a Camera-Saturated Society
   - Smartphones and cheap cameras make it easy for people to record others without consent.
   - This creates concerns around privacy, consent, surveillance anxiety, digital dignity, and image misuse.

4. AI, Synthetic Media, and Trust Collapse
   - AI-generated images and videos may make it difficult to know what is real.
   - This creates both harm and plausible deniability.
   - Fake media can damage people, while real media may be dismissed as fake.

5. Public Understanding of AI
   - AI capability is advancing faster than public understanding.
   - This creates a gap between what technology can do and what people are prepared to understand.

---

## Private raw responses file

A raw responses JSON file was created in the working environment:

`colorful_conversations_raw_responses.json`

It contains raw or near-raw entries and should be treated as internal evidence only.

Important rule:

Do not publish raw responses directly. Use them only to produce public summaries, tags, and graph nodes.

Expected raw response structure:

```json
{
  "id": "rr_001",
  "platform": "reddit",
  "source_type": "comment",
  "source_label": "Reddit thread response",
  "raw_text": "Original response text",
  "speaker": "respondent",
  "handle": "example_handle",
  "timestamp_text": "1h ago",
  "privacy_level": "internal",
  "publishable": false,
  "suggested_theme": "Loss of Privacy in a Camera-Saturated Society",
  "suggested_subthemes": [
    "Smartphone recording",
    "Consent"
  ],
  "notes": "Internal notes"
}
```

---

## Privacy and publication rules

These fields must never be included in the public Kumu feed unless explicitly approved:

- Raw private messages
- Usernames
- Handles
- Phone numbers
- Email addresses
- Source URLs
- Screenshots
- Private analyst notes
- Unapproved quotes
- Sensitive personal details
- Any personally identifying details

Public Kumu data may include:

- Theme names
- Sub-theme names
- Aggregated summaries
- Relationship labels
- Mention counts
- Severity levels
- Public-safe descriptions
- Approved anonymized quotes
- Open questions
- Possible solutions

Default status for all raw responses:

```json
"publishable": false
```

Only public graph records should be exported to Kumu.

---

## Kumu public JSON format

Use this shape for the public Kumu feed:

```json
{
  "elements": [
    {
      "label": "Cost of Living",
      "type": "Theme",
      "cluster": "Economic Pressure",
      "description": "Public-safe summary.",
      "public_note": "Longer public-safe note.",
      "mention_count": 12,
      "severity": "High",
      "status": "published"
    }
  ],
  "connections": [
    {
      "from": "Cost of Living",
      "to": "Debt Stress",
      "type": "contributes to",
      "description": "Rising costs may increase debt pressure.",
      "weight": 8,
      "status": "published"
    }
  ]
}
```

Rules:

- `label` must be unique.
- `from` and `to` must match existing element labels.
- Do not include raw response IDs in the public feed unless they are internal-only and not visible in Kumu.
- Do not include handles, names, or source URLs.
- Keep descriptions concise and public-safe.
- Use consistent relationship types.

Recommended relationship types:

- contributes to
- causes
- worsens
- weakens
- supports
- mitigates
- enables
- protects against
- creates
- affects
- may reduce
- may worsen
- increases urgency of
- outpaces

---

## Suggested implementation

Preferred architecture:

```text
Raw feedback JSON / database
        ↓
Classification and tagging
        ↓
Human review / approval
        ↓
Public graph JSON exporter
        ↓
Kumu import
        ↓
Public read-only problem map
```

For now, a file-based workflow is acceptable.

Recommended files:

```text
/data/raw_responses.json
/data/public_graph.json
/data/themes.json
/data/connections.json
/scripts/build_kumu_graph.py
/scripts/validate_kumu_graph.py
AGENTS.md
README.md
```

If using a database later, keep the same conceptual separation:

- `raw_feedback`
- `public_nodes`
- `public_edges`
- `themes`
- `audit_log`

---

## Codex tasks

Codex should help with the following tasks.

### Task 1: Normalize raw responses

Read the raw responses file and ensure every record has:

- `id`
- `platform`
- `source_type`
- `raw_text`
- `speaker`
- `handle`
- `privacy_level`
- `publishable`
- `suggested_theme`
- `suggested_subthemes`
- `notes`

Do not delete original raw text.

### Task 2: Generate public-safe graph nodes

From raw responses, generate candidate public nodes.

A candidate node should include:

- `label`
- `type`
- `cluster`
- `description`
- `public_note`
- `mention_count`
- `severity`
- `status`

Default status:

```json
"status": "draft"
```

Only human-approved nodes should become:

```json
"status": "published"
```

### Task 3: Generate candidate graph connections

Create edges between related nodes.

Each edge should include:

- `from`
- `to`
- `type`
- `description`
- `weight`
- `status`

Do not create edges to missing nodes.

### Task 4: Validate public graph JSON

Create a validation script that checks:

- JSON is valid.
- Root object has `elements` and `connections`.
- Every element has a unique `label`.
- Every connection has `from`, `to`, and `type`.
- Every `from` and `to` references an existing element.
- No private fields are present in public graph output.
- No raw handles, usernames, phone numbers, or source URLs appear in public graph output.
- No connection points to a draft/private node.

### Task 5: Export Kumu-ready JSON

Create a script that writes:

```text
/data/public_graph.kumu.json
```

This should contain only:

- Published elements
- Published connections
- Public-safe fields

Do not export raw feedback.

### Task 6: Optional CSV export

Kumu can also work with spreadsheet-style data. Optionally export:

```text
/data/kumu_elements.csv
/data/kumu_connections.csv
```

Elements CSV columns:

```text
Label,Type,Cluster,Description,Public Note,Mention Count,Severity,Status
```

Connections CSV columns:

```text
From,To,Type,Description,Weight,Status
```

---

## Coding standards

Use simple, readable code.

Preferred language:

Python for data transformation and validation.

Expected style:

- No unnecessary dependencies.
- Use standard library where possible.
- Keep scripts deterministic.
- Never mutate raw input files unless explicitly requested.
- Write generated files to `/data` or an explicit output path.
- Include clear error messages in validators.
- Fail closed on privacy: if a field looks private, exclude it from public output.

---

## Privacy scanner guidance

When validating public graph JSON, flag values that look like:

- Reddit handles
- Usernames
- Phone numbers
- Email addresses
- URLs
- Source labels that identify a thread or user
- Raw quotes that include personal details

Basic indicators to flag:

- Strings containing `u/`
- Strings containing `reddit.com`
- Strings containing `linkedin.com`
- Strings containing `instagram.com`
- Strings containing `wa.me`
- Strings containing `http://` or `https://`
- Strings matching email patterns
- Strings matching phone-like digit patterns
- Known handles from raw responses, including `Key_Midnight1477` and `srmrox`

The public graph should not include these.

---

## Human review workflow

The system should support this review logic:

1. Raw response enters the system.
2. AI/script suggests themes and subthemes.
3. Candidate nodes and connections are created with `status = "draft"`.
4. Human reviews, edits, and approves.
5. Approved items become `status = "published"`.
6. Exporter generates Kumu JSON using only published items.

Do not auto-publish AI-generated summaries without human review.

---

## Public tone

The public notes should be:

- Clear
- Human
- Non-judgmental
- Not overly academic
- Respectful of respondents
- Focused on patterns rather than blaming individuals
- Careful where issues are sensitive

Avoid:

- Mocking respondents
- Treating raw comments as statistically representative
- Overclaiming based on a small number of responses
- Publishing identifiable details
- Making legal or psychological claims without evidence

Use language such as:

- "A concern raised so far..."
- "One emerging theme is..."
- "This may suggest..."
- "People are describing..."
- "This issue appears connected to..."

---

## Current public graph seed

The current seed graph should include these main elements:

- Rights Without Responsibilities
- Self-Centred Behaviour
- Civic Responsibility
- Social Trust
- Community Discipline
- Individualism
- Public Behaviour
- Lack of Future-Ready Skills
- Digital Literacy
- AI Readiness
- Adaptability
- Education System
- Employment Insecurity
- Reskilling
- Youth Preparedness
- Lifelong Learning
- Loss of Privacy in a Camera-Saturated Society
- Smartphone Recording
- Consent
- Surveillance Anxiety
- Public Privacy
- Image Misuse
- Digital Dignity
- Regulation of Recording
- AI, Synthetic Media, and Trust Collapse
- Deepfakes
- Synthetic Media
- Reputation Harm
- Mental Distress
- Erosion of Trust
- Plausible Deniability
- Media Literacy
- Evidence Uncertainty
- Public Understanding of AI
- AI Capability
- Accountability Problems
- Privacy Harm

---

## Important instruction for future agents

When adding new feedback:

1. Preserve the raw response internally.
2. Do not publish the raw response.
3. Extract one or more themes.
4. Add or update public-safe nodes.
5. Add or update public-safe connections.
6. Keep the Kumu graph focused on conclusions and relationships, not individual comments.
7. Use human approval before public export.

When uncertain, keep data private.

