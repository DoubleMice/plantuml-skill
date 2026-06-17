---
name: plantuml-skill
description: Use when user requests diagrams, flowcharts, sequence diagrams, class diagrams, component diagrams, ER diagrams, architecture charts, or visualizations — including generating a diagram from existing source code, or rendering/extracting PlantUML embedded in a Markdown file to images (e.g. preparing docs for Confluence/Notion). Also use proactively when explaining systems with 3+ components, APIs, data flows, or class hierarchies. Generates .puml files and exports to PNG/SVG with host-installed PlantUML dependencies; public Kroki is used only with explicit user approval.
---

# PlantUML Diagram Skill

## Overview

Generate `.puml` PlantUML diagram files and export to PNG/SVG using host-installed PlantUML dependencies. Prefer the local `plantuml` CLI, then a local `plantuml.jar`. Use public Kroki only when the user explicitly allows uploading the `.puml` source.

**Format:** `.puml` (PlantUML text)
**Renderer:** host `plantuml` CLI or host `plantuml.jar`; public Kroki requires explicit approval
**Output:** PNG, SVG
**Diagram types:** sequence, component, class, ER, activity, use case, state, C4, and more

## When to Use

**Explicit triggers:**
- "plantuml diagram", "sequence diagram", "class diagram", "component diagram"
- "UML", "activity diagram", "use case diagram", "state machine"
- "visualize", "draw", "diagram", "flowchart", "architecture chart"

**Proactive triggers:**
- Explaining a system with 3+ interacting components
- Describing API flows, authentication sequences, message passing
- Showing class hierarchies, database schemas, or ER models
- Illustrating state machines or lifecycle flows

**When NOT to use it — route elsewhere:**
- General, non-UML quick diagrams embedded in Markdown → **mermaid**.
- Freeform, heavily-styled, or branded diagrams needing pixel control → **drawio**.
- A hand-drawn / sketchy look → **excalidraw** or **tldraw**.

## Modes

Once triggered, route by what the user actually wants — then run the shared render loop (Steps 4–8):

| Mode | The user wants… | Entry point |
|---|---|---|
| **Generate** (default) | a diagram from a text description | Steps 1–8 below |
| **From code** | a diagram of existing source code | [`references/from-source-code.md`](references/from-source-code.md) → Steps 4–8 |
| **Embed** | the PlantUML inside a Markdown doc rendered to images | [`references/markdown-embed.md`](references/markdown-embed.md) |
| **Refine** | to change an existing diagram | load its `.puml`, apply the minimal edit (Step 7), re-render (Steps 4–6) |
| **Review** | to know whether an existing diagram is readable / correct | run the Step 6 vision self-check on the image |

## Prerequisites

### Dependency policy

This skill is instruction-only at install time. It must not install runtime
dependencies by itself. Rendering dependencies are optional and backend-specific:

- `.puml` generation: no runtime dependency.
- Host PlantUML CLI rendering: `plantuml` and Graphviz (`dot`) installed on the host.
- Host PlantUML jar rendering: Java, Graphviz (`dot`), and `plantuml.jar`.
- Public Kroki rendering: `curl` and explicit user approval because the diagram
  source leaves the machine.

Installing this skill only installs instruction files under the Codex skills
directory. It does not install Java, Graphviz, `plantuml`, or `plantuml.jar`.
Do not install host packages or contact public renderers unless the user asks
for that action or explicitly approves it for the current task.

Generating `.puml` text has no external runtime dependency. Rendering to PNG/SVG
requires one of the backends below:

**Option A: Host PlantUML CLI (recommended)**
```bash
# macOS
brew install plantuml graphviz

# Ubuntu / Debian
sudo apt-get update
sudo apt-get install -y plantuml graphviz

plantuml -version
dot -V
```
This backend runs on the host and keeps PlantUML source on the machine.

**Option B: Host PlantUML jar**
```bash
# Requires Java + Graphviz + plantuml.jar on the host.
brew install graphviz   # macOS
sudo apt-get install -y graphviz  # Ubuntu / Debian
# Download plantuml.jar from https://plantuml.com/download
PLANTUML_JAR=/path/to/plantuml.jar
java -jar "$PLANTUML_JAR" -version
```
This backend runs fully local once the jar and dependencies are already present.

**Option C: Public Kroki (only after explicit user approval)**
```bash
# Requires curl and sends .puml source to a third-party service.
curl --version
```

## Workflow

### Step 1: Check Dependencies
```bash
plantuml -version     # preferred host renderer
dot -V                # Graphviz layout engine
java -version         # only needed for direct plantuml.jar use
curl --version        # only needed for approved public Kroki use
```
Decide the renderer before creating output. Prefer the host `plantuml` CLI. If
it is unavailable, use a host `plantuml.jar` only when Java, Graphviz, and the
jar path are available. If no host renderer is available, generate only the
`.puml` source and report that rendering was skipped. Do not contact public
Kroki unless the user explicitly approved it for this task.

### Step 2: Pick Diagram Type
Choose the most appropriate PlantUML diagram type (see reference below).

### Step 3: Generate .puml File
Write the PlantUML source file with `@startuml` / `@enduml` markers.

### Step 4: Export via the selected host backend (capture errors)
Pick the backend first and use one backend throughout. The default below uses
the host `plantuml` CLI. See [`references/rendering-backends.md`](references/rendering-backends.md).
If the host CLI and host `plantuml.jar` are unavailable, stop after writing the
`.puml` file unless the user explicitly approves public Kroki.

```bash
# PNG (recommended)
plantuml -tpng diagram.puml

# SVG
plantuml -tsvg diagram.puml
```

For a host PlantUML jar, use `java -jar "$PLANTUML_JAR" -tpng diagram.puml`
or `java -jar "$PLANTUML_JAR" -tsvg diagram.puml`, then run the same file
validation in Step 5. Public Kroki (`https://kroki.io`) is allowed only after
explicit user approval; when used, report that the source was uploaded.

### Step 5: Validate & self-correct (loop — do NOT skip)
Never report success on a blind renderer command. Verify the output first; treat the export as **failed** if any of these hold:
- The renderer command exited non-zero.
- The file is empty: `[ -s diagram.png ]` fails.
- The bytes aren't a real image: `file diagram.png` should report `PNG image data`; for SVG the file should start with `<svg` or `<?xml`.

```bash
if [ ! -s diagram.png ] || ! file diagram.png | grep -q "PNG image data"; then
  echo "Render failed or output was not a PNG image"
fi
```

On failure: read the renderer stderr/stdout, fix the flagged `.puml` line (see
**Common Mistakes**), then re-run Step 4. **Repeat up to 3 times.** If a
targeted line fix doesn't clear it, degrade in this order, re-rendering after
each step — stop as soon as it renders:

1. remove exotic shapes → plain `rectangle`/`component`/`node`
2. strip `skinparam` / `!theme` (render plain first)
3. remove `note` lines
4. simplify labels, wrap in `"…"`
5. reduce edges
6. switch to a simpler diagram type rather than forcing the current one

For a per-diagram-type error catalog and the PlantUML safe subset, read [`references/plantuml-troubleshooting.md`](references/plantuml-troubleshooting.md). If it still fails after 3 tries, stop and show the user the raw renderer error — do not claim the diagram was produced.

### Step 6: Self-check (vision)

The Step 5 loop only proves the renderer returned a **valid image** — not that the diagram is **readable**. After it renders, use the agent's vision capability to read the PNG and catch what auto-layout (Graphviz) can't prevent. PlantUML positions everything itself, so the failures here are about readability, not your coordinates:

| Check | What to look for | Fix |
|---|---|---|
| Label truncation / overrun | Text clipped or spilling past a box | Shorten the label, wrap in `"…"`, or break with `\n` |
| Component overlap / cramped | Boxes touching or crowded; unreadable | Add `together { }`, layout hints, or split the diagram |
| Wrong orientation / aspect | Diagram far too wide or too tall to read | Switch `left to right direction` ↔ `top to bottom direction` |
| Edge spaghetti | Many relations crossing, hard to follow | Reorder declarations, group with `package`/`together`, or add hidden edges for layout |
| Wrong diagram type | Type doesn't suit the content | Switch type (sequence, state, C4, …) |
| Low contrast | Text blends into the fill / theme | Adjust `skinparam` / `!theme` so text contrasts the fill |

- Max **2 self-check rounds** — if issues remain after 2 fixes, show the user anyway.
- **Re-render (Step 4) and re-validate (Step 5) after every fix.**
- If vision is unavailable, skip self-check and show the PNG directly.

### Step 7: Review loop

After self-check, show the exported image and collect feedback. Apply the **minimal `.puml` edit** for each request, then re-render and re-validate:

| User request | Edit action |
|---|---|
| Change a label | Edit the element / message text in the `.puml` |
| Add / remove an element or relation | Add or delete the matching line |
| Change a color | `skinparam`, `!theme`, or an inline `#color` on the element |
| Change layout direction | Swap `left to right direction` ↔ `top to bottom direction` |
| Restructure / group | Wrap related elements in a `package` / `together { }`, or regenerate |

- Overwrite the same `diagram.puml` / output file each round — don't create `v1`, `v2`, …
- **Safety valve:** after 5 rounds, suggest the user fine-tune the `.puml` directly or with the configured local renderer.

### Step 8: Report to User
Only after Steps 5–7 pass. Tell the user:
- Path to the `.puml` source file
- Path to the exported PNG/SVG
- Brief description of what was generated
- Which backend rendered it, and whether the source left the machine — e.g. "via host PlantUML CLI (stayed local)", "via host PlantUML jar (stayed local)", or "via public Kroki (uploaded to kroki.io after approval)"

---

## Import Workflows

Two non-default modes — load the linked playbook when triggered, then run the same Step 4–8 loop:

- **Generate a diagram from existing source code** — class diagram of a module, sequence from a request handler, component map of a repo, ER from ORM models. Read the code, extract the real entities/relationships, draw only what's there. → [`references/from-source-code.md`](references/from-source-code.md)
- **Render PlantUML embedded in Markdown** — extract ` ```plantuml ` / ` ```puml ` blocks (and linked `.puml`), render each to an image, and rewrite the Markdown with image links (e.g. to publish to Confluence / Notion, which don't render fenced PlantUML). → [`references/markdown-embed.md`](references/markdown-embed.md)

---

## Diagram Types

| Type | Keyword | Use for |
|------|---------|---------|
| Sequence | `@startuml` + sequence syntax | API calls, protocol flows, message passing |
| Component | `@startuml` + components | service architecture, module dependencies |
| Class | `@startuml` + class syntax | OOP models, data structures |
| ER / Entity | `@startuml` + entity syntax | database schemas |
| Activity | `@startuml` + activity syntax | workflows, business processes |
| Use Case | `@startuml` + actor/usecase | system requirements, user stories |
| State | `@startuml` + state syntax | state machines, lifecycle |
| C4 Context | `@startuml` + C4 includes | high-level system context maps |
| Mind Map | `@startmindmap` | topic breakdowns, concept maps |
| Gantt | `@startgantt` | project timelines, schedules |

---

## Syntax Reference

### Component / Architecture Diagram

```plantuml
@startuml
!theme plain

title Microservices Architecture

actor "Client" as client
rectangle "API Gateway" as gateway #LightBlue

rectangle "Services" {
  component "User Service" as user
  component "Order Service" as order
}

database "User DB" as userdb
database "Order DB" as orderdb
queue "Kafka" as kafka

client --> gateway
gateway --> user
gateway --> order
user --> userdb
order --> orderdb
order --> kafka : events

@enduml
```

**Shape types:**
- `actor "Name" as id` — stick figure (user, external actor)
- `component "Name" as id` — component box with [brackets]
- `rectangle "Name" as id` — plain rectangle (for groups/layers)
- `database "Name" as id` — cylinder (database)
- `queue "Name" as id` — queue symbol
- `cloud "Name" as id` — cloud shape (external services)
- `node "Name" as id` — server/node box
- `frame "Name" as id` — frame grouping
- `package "Name" { }` — package grouping

**Arrows:**
- `A --> B` — solid arrow
- `A -> B` — thin arrow
- `A ..> B` — dashed arrow
- `A --> B : label` — labeled arrow
- `A <--> B` — bidirectional

**Colors:**
- `#LightBlue`, `#LightGreen`, `#LightYellow`, `#Pink`, `#Violet`
- `#AED6F1` (blue), `#A9DFBF` (green), `#FAD7A0` (orange), `#F1948A` (red)
- `#D7BDE2` (purple), `#F9E79F` (yellow), `#D3D3D3` (grey)

---

### Sequence Diagram

```plantuml
@startuml
!theme plain
title Login Flow

participant "Client" as C
participant "API Gateway" as G
participant "Auth Service" as A
database "User DB" as D

C -> G : POST /login
G -> A : validateCredentials(user, pass)
A -> D : SELECT * FROM users WHERE email = ?
D --> A : user record
A --> G : 200 OK + JWT token
G --> C : { token: "..." }

@enduml
```

**Arrow types:**
- `A -> B` — synchronous call
- `A --> B` — return / dashed
- `A ->> B` — async message
- `A -[#red]-> B` — colored arrow
- `activate A` / `deactivate A` — show activation box

---

### Class Diagram

```plantuml
@startuml
!theme plain

class User {
  +int id
  +String name
  +String email
  +login() : bool
  +logout()
}

class Order {
  +int id
  +Date createdAt
  +float total
  +place()
  +cancel()
}

class Product {
  +int id
  +String name
  +float price
}

User "1" --> "*" Order : places
Order "*" --> "*" Product : contains

@enduml
```

**Relationships:**
- `A --> B` — association
- `A --|> B` — inheritance
- `A ..|> B` — implements interface
- `A *-- B` — composition
- `A o-- B` — aggregation
- `A "1" --> "*" B : label` — with multiplicities

---

### ER Diagram

```plantuml
@startuml
!theme plain

entity "USER" as user {
  * id : int <<PK>>
  --
  name : varchar
  email : varchar
  created_at : datetime
}

entity "ORDER" as ord {
  * id : int <<PK>>
  --
  * user_id : int <<FK>>
  total : decimal
  status : varchar
}

entity "PRODUCT" as prod {
  * id : int <<PK>>
  --
  name : varchar
  price : decimal
}

user ||--o{ ord : places
ord }o--|{ prod : contains

@enduml
```

---

### Activity / Flowchart

```plantuml
@startuml
!theme plain

start

:Receive Order;

if (Payment valid?) then (yes)
  :Process Payment;
  :Send Confirmation Email;
  :Update Inventory;
  :Ship Order;
  :Mark as Delivered;
else (no)
  :Send Payment Failed Email;
  :Cancel Order;
endif

stop

@enduml
```

---

### State Diagram

```plantuml
@startuml
!theme plain

[*] --> Pending

Pending --> Processing : payment_received
Processing --> Shipped : packed
Shipped --> Delivered : confirmed
Processing --> Cancelled : cancel
Pending --> Cancelled : cancel

Delivered --> [*]
Cancelled --> [*]

@enduml
```

---

### C4 Context Diagram

C4 uses the bundled C4-PlantUML standard library via `!include <C4/...>`, which recent host PlantUML installations resolve with no network fetch.

```plantuml
@startuml
!include <C4/C4_Context>

title System Context — Internet Banking

Person(customer, "Banking Customer", "A customer of the bank")
System(banking, "Internet Banking System", "Lets customers view their accounts")
System_Ext(mail, "E-mail System", "The internal Microsoft Exchange system")

Rel(customer, banking, "Uses", "HTTPS")
Rel(banking, mail, "Sends e-mail via", "SMTP")
@enduml
```

Other levels: `<C4/C4_Container>` (`Container`, `ContainerDb`), `<C4/C4_Component>` (`Component`). Common macros: `Person`, `System`, `System_Ext`, `Container`, `Rel`, `Boundary`. **Do not** use a remote `!includeurl https://…`; always use the bundled `<C4/…>` form.

---

## Export Commands

Quick reference for the renderer variants. When actually exporting, use the Step 4 form and run the Step 5 validation loop.

```bash
# PNG via host PlantUML CLI (recommended)
plantuml -tpng diagram.puml

# SVG via host PlantUML CLI
plantuml -tsvg diagram.puml

# Via host PlantUML jar
java -jar "$PLANTUML_JAR" -tpng diagram.puml

# Via public Kroki only after explicit approval
curl -s -X POST https://kroki.io/plantuml/png \
  -H "Content-Type: text/plain" \
  --data-binary "@diagram.puml" \
  -o diagram.png
```

---

## Themes

```plantuml
!theme plain       ← clean, minimal (recommended)
!theme cerulean    ← blue-tinted
!theme blueprint   ← dark blue background
!theme aws-orange  ← AWS style
!theme vibrant     ← vivid colors
```

Or use `skinparam` for custom styling:
```plantuml
skinparam backgroundColor #FAFAFA
skinparam componentBorderColor #555555
skinparam ArrowColor #333333
skinparam FontName Arial
```

---

## Common Mistakes

Quick table below; for a per-diagram-type error catalog, the PlantUML safe subset, and the failure-degradation ladder, see [`references/plantuml-troubleshooting.md`](references/plantuml-troubleshooting.md).

| Mistake | Fix |
|---------|-----|
| `plantuml` command not found | Install host dependencies, then verify with `plantuml -version` and `dot -V` |
| Renderer exits with syntax error | Read stderr/stdout, fix the offending line, and re-render via the Step 5 loop |
| Public Kroki returns 400 Bad Request | `cat` the output file — Kroki wrote the offending line + reason there; fix it and re-render via the Step 5 loop |
| Arrow direction unexpected | Use `-->` for downward/right; explicitly use `-up->`, `-down->`, `-left->`, `-right->` |
| Diagram too large/crowded | Split into multiple diagrams or use `package`/`rectangle` grouping |
| Missing `@startuml` / `@enduml` | Always wrap diagram in these markers |
| Special chars in labels | Wrap in quotes: `"Label: value"` |
| C4 includes not found | Use the bundled `!include <C4/C4_Context>`; never a remote `!includeurl https://…` |
| Component overlap | Use `together { }` or explicit layout hints (`top to bottom direction`) |
| Sequence participants out of order | Declare `participant` explicitly at top in desired left-to-right order |
