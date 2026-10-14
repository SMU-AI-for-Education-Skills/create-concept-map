---
name: concept-map
description: >
  Generates an interactive, force-directed concept map as a self-contained HTML file from any
  academic or educational text. Use this skill whenever the user wants to visualise the key
  concepts and relationships in a paper, article, book chapter, or document — even if they
  don't use the word "concept map". Trigger phrases include: "concept map", "map the ideas in",
  "visualise this paper", "show the concepts in", "make a concept map", "turn this into a map",
  "diagram the ideas", "interactive diagram from this text", or any request to produce a
  student-facing visual summary of a reading. Always use this skill when the input is academic
  or educational content and the goal is visual understanding, especially for distribution to
  students.
---

# Concept Map Skill

Produces a downloadable, self-contained interactive HTML concept map from academic text.
The map uses a force-directed layout with expandable node tiers, hover definitions, a
navigation help panel, a student-side editor, and a two-page print view. It is designed to be
distributed to students as a single file that works offline in any modern browser.

This skill specifies *what* the map must contain and *how it must behave*. It deliberately
does not dictate the code. Write the HTML, CSS and JavaScript yourself, to the specification
below. The specification exists because each feature serves a teaching purpose (explained
alongside it); keep those purposes in view when making implementation choices.

---

## Step 1 — Read the input

Determine the input type and extract the text:

- **Pasted text**: use directly.
- **Uploaded PDF**: follow `/mnt/skills/public/pdf-reading/SKILL.md` to extract text.
- **Uploaded DOCX**: follow `/mnt/skills/public/docx/SKILL.md` to extract text.
- **URL (web article)**: use `web_fetch` to retrieve the page content.
- **URL (PDF)**: use `web_fetch` with `web_fetch_pdf_extract_text: true`.

If the user specifies a subsection (e.g. "the third chapter" or "the section on X"), focus
extraction and concept selection on that portion only.

---

## Step 1.5 — Choose map mode (ask before generating)

Before extracting concepts, ask the instructor which map mode they want. Use this exact
message, replacing nothing — it is shown verbatim to the instructor:

---

**Before I build the map — what kind of output would you like?**

**Comprehensive** — a complete concept map with all nodes, relationships, and descriptions.
Best for: introducing a topic, providing reference material, or post-lecture review.

**Skeleton — reduced depth** — only tier-0 and tier-1 nodes are included; tier-2 and tier-3
nodes are omitted entirely. Students see the spine of the argument and must elaborate it
themselves. Best for: consolidation and recall activities.

**Skeleton — category omission** — the full node structure is included, but all nodes
belonging to one chosen category are replaced with blank placeholder cards (label hidden,
description replaced with a fill-in prompt). Students must identify *what type of concept
is missing* and supply examples. Best for: higher-order tasks requiring students to notice
and reason about absence.

**Skeleton — hidden descriptions** — all nodes and edges are visible, but every tooltip
description is replaced with a blank fill-in prompt. Students must supply definitions.
Best for: pre-lecture priming or vocabulary consolidation.

Which would you like?

---

**If the instructor chooses "category omission"**: before proceeding, do a quick first-pass
scan of the text to identify which type categories will appear in the map (e.g. for a
philosophy text: Core concept, Structural concept, Thinker/source, Foil/critique,
Implication). Then ask:

> "I can see this text will have nodes in these categories: **[list them]**. Which category
> would you like to omit? Omitting **Foils/critiques** works well for argumentation tasks;
> omitting **Thinkers** tests historical-contextual knowledge; omitting **Implications**
> suits synthesis tasks."

Wait for the instructor's answer before proceeding to Step 2. Record the chosen mode and
(if applicable) the chosen omission category — you will need them in Step 3.

**Mutual exclusivity**: only one skeleton mode may be active at a time. Do not combine
reduced depth with category omission.

---

## Step 2 — Extract concepts and relationships

Read the text carefully before building the graph. Do not keyword-extract — read for meaning.

### Node schema

Each node has:
- `id`: short camelCase string, unique
- `label`: display name, 1–4 words (use the text's own terminology where possible)
- `tier`: 0 (root), 1 (primary), 2 (secondary), 3 (peripheral)
- `type`: one of the type categories below — choose based on role in the text
- `desc`: 1–3 sentence tooltip definition, written for a student reader

### Tier guidelines

| Tier | Role | Count |
|------|------|-------|
| 0 | The single central thesis, concept, or subject of the text | exactly 1 |
| 1 | Core concepts directly constituting or defining the tier-0 node | 4–8 |
| 2 | Supporting concepts, mechanisms, critiques, thinkers, applications | 6–14 |
| 3 | Peripheral detail, downstream implications, named examples | 4–10 |

Total nodes: aim for 18–30. Fewer for short texts; more for dense book chapters.

### Type categories

Choose the type category that best describes the node's *function* in the argument, not its
surface form. Use consistent types across the map so the colour legend is meaningful.

Common patterns by discipline:

**Philosophy / humanities:**
- `core` — central thesis or concept
- `struct` — structural/analytical concept the argument depends on
- `thinker` — philosopher or theorist drawn upon
- `foil` — position the text argues against
- `impl` — implication or consequence

**Science / NLP / AI:**
- `core` — central claim or system
- `method` — technique or algorithm
- `threat` / `risk` — problem or failure mode
- `solution` — proposed remedy or intervention
- `domain` — field or application area

**Social science / education:**
- `core`, `concept`, `theory`, `factor`, `outcome`, `context`

You may define your own types if none of these fit — just ensure 2–5 distinct types appear so
the legend is useful and the colour coding carries meaning.

### Edge schema

Each edge has:
- `s`: source node id
- `t`: target node id
- `r`: relationship label, 2–4 words (verb phrase: "leads to", "depends on", "critiques", etc.)

Edges should express meaningful directed relationships, not just co-occurrence. Ask: what does
the source *do to* or *say about* the target?

### Compression vs expansion decisions

- **Compress** a concept to a single node when the text treats it as a given or names it
  without developing it. Example: "Heidegger" in a Gadamer map.
- **Expand** a concept into its own subgraph (tier 1 node with tier 2 children) when the text
  devotes substantial argument to unpacking it.
- When in doubt about scope: a dense book chapter warrants deeper expansion than a 10-page
  article. Match depth to the text's own emphasis.

---

## Step 3 — Apply the chosen mode to the data

Transform the extracted nodes and edges according to the mode chosen in Step 1.5 *before*
building the page. The page records the mode so it can explain it to students on screen and
in print.

**Comprehensive** — no changes. Mode is `"none"`.

**Reduced depth** — keep only nodes with `tier <= 1`, and only edges whose source and target
both survive. Mode is `"depth"`.

**Category omission** — keep all nodes and edges. For every node whose `type` matches the
omitted category, set `label` to `"?"`, set `desc` to
`"This node is intentionally hidden. Can you identify what concept belongs here and why?"`,
and add `skeleton: true`. Mode is `"category"`, and record the omitted category's display
name (e.g. `"Foils / critiques"`).

**Hidden descriptions** — keep all nodes and edges. Replace every `desc` with
`"[Your definition here — what does this concept mean and what role does it play in the argument?]"`.
Mode is `"hidden-desc"`.

---

## Step 4 — Build the HTML page

Write one self-contained HTML file. The sections below describe what it must do. How you
structure the code is up to you, provided the result meets every requirement here and in the
quality checklist.

### Technical constraints

- **Single file.** All CSS and JavaScript inline. The only permitted external dependency is a
  graph/visualisation library (such as D3) loaded from `https://cdnjs.cloudflare.com` at a
  pinned version. Students receive this as one file, often by email or LMS upload, so nothing
  may rely on sibling files or a server.
- **Data as plain literals.** Declare the graph data near the top of the main script as two
  mutable array literals: `let NODES = [...];` and `let EDGES = [...];`, using the schemas
  from Step 2. Also declare the mode and omitted category as constants
  (e.g. `SKELETON_MODE`, `SKELETON_OMITTED`). Keeping data in this predictable form is what
  makes both the in-page "Save my version" feature and instructor edits in Step 6 reliable.
- **Title.** A short descriptive map title appears in both `<title>` and a visible heading,
  e.g. "Gadamer — Historically Effected Consciousness".
- **No browser storage** is required; edits live in memory and are persisted by downloading
  a new copy of the file.

### Layout

- The map canvas is the dominant element and fills the available viewport height. A narrower
  sidebar (roughly 280px on desktop) holds controls, the legend, the navigation help, and the
  editor. The map should never be squeezed to make room for the sidebar's content — the
  sidebar scrolls instead.
- On narrow screens (phones), stack the sidebar below the map rather than shrinking the map
  to illegibility. Students frequently open distributed files on phones.
- Use a calm, neutral background with the map canvas visually set apart (subtle border,
  slightly different tint, rounded corners). The visual emphasis should be on the nodes.
- If the user's environment supports it, respecting `prefers-color-scheme: dark` is welcome,
  but print output must always be light.

### Visual encoding

- **Nodes** are rounded rectangles containing their label. Size and font scale with tier so
  the hierarchy is legible at a glance: the root is clearly largest and bold-stroked; tier 3
  is smallest. (Roughly a 2:1 ratio between root and tier-3 size works well.)
- Wrap labels across up to three lines so they stay inside the node.
- **Colour by type.** Each type gets a light fill and a darker stroke (also used for the
  label text). Use these palettes:

| Suggested use | Light fill | Stroke | Dark-mode fill |
|---------------|-----------|--------|-----------|
| core / central | `#EEEDFE` | `#534AB7` | `#AFA9EC` |
| struct / method / theory | `#E1F5EE` | `#0F6E56` | `#5DCAA5` |
| thinker / reference | `#E6F1FB` | `#185FA5` | `#85B7EB` |
| foil / threat / risk | `#FAECE7` | `#993C1D` | `#F0997B` |
| impl / domain / outcome | `#FAEEDA` | `#854F0B` | `#EF9F27` |

  If you introduce a sixth type, use: light `#FBEAF0` / stroke `#993556` / dark `#ED93B1`.
- **Skeleton placeholders** (category omission) use a neutral grey — fill `#F5F5F3`, stroke
  `#B0AEA8`, dashed outline — rather than their type colour, because the type *is* the answer.
- **Edges** are directed lines with a small arrowhead stopping at the target node's edge,
  drawn in a muted grey at low opacity so they don't overpower the nodes.
- **Edge labels** (the relationship verb phrases) are always visible at the midpoint of each
  edge, in small muted text. Relationships are the substance of a concept map; hiding them
  behind interaction would defeat its purpose.
- **Legend** listing each type actually used in the map, with a colour swatch and a
  human-readable name (e.g. "Thinker / source", not `thinker`).

### Layout engine

- Use a force-directed layout tuned so the root sits near the centre, higher tiers repel more
  strongly, links between higher-tier nodes are longer, and nodes never overlap (collision
  padding beyond each node's size).
- Compute the layout to completion before display and then hold node positions fixed. The map
  should appear settled, not jitter or drift, because students will be reading it.
- Positions persist: when the visible set changes (expand/collapse, depth change, edits),
  already-placed nodes stay where they were, and nodes a student has dragged stay where they
  were dropped.

### Interactions

Each of these must work and be listed in the on-page navigation help panel, with the action
shown as a small key-style tag next to a plain-language description:

| Action | Behaviour |
|--------|-----------|
| Click node | Highlight its neighbourhood: the node is emphasised, directly connected nodes and their edges stay prominent, everything else fades. Click again, or click the background, to clear. A "Clear highlight" control appears while a highlight is active. |
| Double-click node | Expand or collapse its children. Must not also trigger the single-click highlight. |
| Hover node | Show a tooltip with the label (bold) and its description, positioned near the cursor and kept inside the canvas. |
| Hover node → pencil icon | A small ✎ button appears at the node's top-right corner; clicking it opens the editor on that node (see Editor). |
| Drag node | Reposition it; connected edges and labels follow live; the node stays where dropped. |
| Scroll | Zoom in and out (sensible limits, roughly 0.3× to 2.5×). |
| Drag background | Pan the map. |

### Controls

Provide buttons for: **Reset view** (return zoom/pan to default), **Re-layout** (discard all
saved and dragged positions and recompute the layout), **Expand all**, **Collapse** (to root
only), a **Depth** slider from 1 to 3 with its current value shown (default 2), and
**🖨 Print**.

Initial state: tier-0 and tier-1 nodes expanded, depth 2. A node is visible when its tier is
within the current depth and it is the root, is itself expanded, or is directly connected to
an expanded node.

### Skeleton banner

When the mode is anything other than `"none"`, show a clearly visible banner above the map
(warm amber tone works well) explaining the task to the student:

- **Reduced depth**: only the core concepts are shown; use the editor to add the supporting
  concepts you identify from your reading.
- **Category omission**: all nodes in the *[omitted category]* category are hidden as grey "?"
  cards; identify what concept belongs in each gap, and why, and use the editor to fill them in.
- **Hidden descriptions**: the structure is complete but definitions are removed; write your
  own description for each node via the editor before checking against the source.

### Editor (student-side)

The editor lets students work on the map itself — completing skeletons, adding their own
connections, correcting or annotating. It lives in the sidebar, hidden behind an "✏ Edit map"
toggle button by default so the map stays uncluttered for students who only want to read.

It must support:

- **Node list** — every node with its label (or "?" for placeholders), type and tier, and
  Edit / Delete buttons. The root cannot be deleted. Deleting a node asks for confirmation and
  removes its edges.
- **Edit node** — fields for label, description, type and tier. Opened from the node list or
  the pencil icon. While editing, show a banner naming the node being edited (tinted in that
  node's type colour) and mark the node on the map with a distinct dashed ring, so the student
  can see which node they're changing. Save and Cancel both clear the ring and banner; saving
  a placeholder node removes its placeholder status. Focus the label field on open.
- **Edge list** — each edge shown as "Source → relationship → Target" with a Delete button.
- **Add node** — label (required), description, type, tier (default 2). Brief success or
  validation message.
- **Add edge** — choose source and target from the existing nodes, enter a relationship
  label (required); source and target must differ. Brief success or validation message.
- **💾 Save my version** — downloads a copy of the complete page with the current nodes and
  edges written back into the `NODES` and `EDGES` literals, named after the map title with an
  `-edited` suffix. Include a one-line explanation that opening the downloaded file lets them
  continue where they left off. This is how students keep their work, so the downloaded file
  must open and behave exactly like the original apart from the changed data.

Every edit re-renders the map immediately.

### Print view

Printing produces a two-page handout, because students and instructors often want paper copies
for annotation or in-class activities:

- **Page 1 — the map as currently displayed.** A static snapshot of the visible nodes and
  edges, with the title above it. Fit it to the page using a tight bounding box around the
  nodes (independent of the on-screen zoom/pan), with all edge labels visible and no
  highlight or fade effects.
- **Page 2 — concept definitions.** A meta line (map title, number of concepts and
  relationships), the skeleton note if applicable, then every node sorted by tier and then
  alphabetically, in two columns. Each entry shows the label in its type's stroke colour, a
  small uppercase tier/type line (e.g. "Primary concept · thinker"), and the description, with
  a coloured left border. Entries do not split across columns or pages.
  - Category-omission placeholders read "? (hidden concept)" with a grey border and a blank
    ruled space prompting "What concept belongs here? Why?".
  - In hidden-descriptions mode, every entry shows a blank ruled space prompting "Define this
    concept in your own words:" instead of the description.

The 🖨 Print button prepares both pages (stop any layout motion, build the snapshot and
definitions) and then opens the browser's print dialog. The interactive interface is hidden
in print.

---

## Step 5 — Output

Save the completed HTML to `/mnt/user-data/outputs/<slug>.html` where `<slug>` is a
kebab-case version of the map title. Then call `present_files` with that path.

Do not display the HTML inline in chat — it is a standalone file for distribution.
After presenting the file, write a brief 2–3 sentence summary of the map's structure
(how many nodes, tiers, the main conceptual clusters). Then offer the instructor
structured editing guidance — paste the following block verbatim, substituting actual
node labels from the map you just generated into the examples:

---

**Want to adjust the map before sharing it?** Here's what I can change:

- **Add a node** — e.g. "Add a tier-2 node called 'X' connected to '[node]' with the relationship 'grounds'"
- **Remove a node** — e.g. "Remove the node '[node]' and its edges"
- **Edit a label or description** — e.g. "Rename '[node]' to '…'" or "Update the tooltip for '[node]' to say …"
- **Change a node's tier** — e.g. "Promote '[node]' to tier 1"
- **Add an edge** — e.g. "Add an edge from '[node A]' to '[node B]' labelled 'depends on'"
- **Remove an edge** — e.g. "Remove the edge between '[node A]' and '[node B]'"
- **Change an edge label** — e.g. "Change the edge from '[node A]' to '[node B]' to say 'contrasts with'"

Just describe what you'd like and I'll update the file.

---

## Step 6 — Instructor edits (if requested)

If the instructor asks for changes to a previously generated map, read the existing HTML
file from `/mnt/user-data/outputs/`, locate the `NODES` and `EDGES` array literals, apply the
requested changes directly to those arrays using `str_replace`, save to the same path, and
call `present_files` again. Do not regenerate the whole map from scratch — preserve all
existing nodes, edges, ids, and the page's code and styling. Confirm the change in one
sentence after presenting the file.

---

## Quality checklist

Before saving, verify:

**Content**
- [ ] Exactly one tier-0 node; total nodes 18–30 (reduced-depth skeleton may be fewer)
- [ ] Every non-skeleton node has a non-empty `desc` written for a student reader
- [ ] Edge labels are verb phrases (2–4 words), not nouns
- [ ] 2–5 distinct types; every type used has a colour, and the legend matches the types actually used
- [ ] The chosen mode is applied correctly (for category omission: every node of the omitted
      type has `skeleton: true`, label `"?"`, and the fill-in `desc`)

**Structure**
- [ ] One self-contained file; the only external resource is a pinned library from cdnjs
- [ ] `let NODES` and `let EDGES` are plain array literals near the top of the main script
- [ ] Mode and omitted category recorded as constants and reflected in the banner and print view
- [ ] Map title in both `<title>` and the visible heading

**Behaviour**
- [ ] Layout settles before display; positions persist across re-renders and drags; Re-layout resets them
- [ ] Edge labels always visible
- [ ] Click-highlight, double-click expand/collapse, hover tooltip, pencil-to-edit, drag, zoom and pan all work, and all are listed in the help panel
- [ ] Reset view, Re-layout, Expand all, Collapse, Depth slider and Print controls all work
- [ ] Editor supports edit/delete/add for nodes and edges, with the editing ring and banner
- [ ] Save my version downloads a working copy with the edits written into the data literals
- [ ] Print gives page 1 map snapshot and page 2 two-column definitions, with correct skeleton variants
- [ ] Usable on a phone-width screen (sidebar stacks below the map)

Before presenting, sanity-check the file: confirm the script has no syntax errors (e.g. extract
it and run `node --check`), and that the NODES/EDGES literals parse and every edge references
existing node ids.
