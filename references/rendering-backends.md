# Rendering Backends — Selection, Precedence & Transparency

The skill can render through host backends by default. Public Kroki is retained
only as an explicit-approval fallback. The backends differ in **install cost**
and, crucially, **where your source goes**.

| Backend | Endpoint / command | Dependencies | Source leaves your machine? |
|---|---|---|---|
| **Host PlantUML CLI** (default) | `plantuml -tpng diagram.puml` / `plantuml -tsvg diagram.puml` | Host `plantuml` and Graphviz (`dot`). | No |
| **Host `plantuml.jar`** | `java -jar "$PLANTUML_JAR" -tpng diagram.puml` | Host Java, Graphviz, and `plantuml.jar`. | No |
| **Public Kroki** (explicit approval only) | `https://kroki.io/plantuml/{png,svg}` | `curl` and explicit user approval. | **Yes** — POSTed to a third-party service |

Installing the skill does not install any of these dependencies. When no host
renderer is already available, write the `.puml` source and stop instead of
installing dependencies or contacting a public service.

## Selection / precedence

Decide the backend *before* Step 4:

1. **Honor an explicit host choice** — if the user named the host `plantuml`
   CLI or a host jar path, use it.
2. **Try host PlantUML CLI** — default to `plantuml` when it is available.
3. **Try host `plantuml.jar`** — use it when Java, Graphviz, and the jar are
   available.
4. **No host renderer** — write the `.puml` source and report that rendering was
   skipped. Do not send source to public Kroki as a fallback.
5. **Public Kroki** — use `https://kroki.io` only when the user explicitly
   requests or approves it for this task.

Use one renderer throughout a session; don't mix.

## No silent downgrade or network fallback

This is the rule that makes the choice trustworthy:

- **Don't quietly fall back.** If the chosen backend is unavailable (`plantuml`
  missing, no `java`/jar, or `dot` missing), do *not* silently switch to another.
  Tell the user which backend failed and what the alternatives are. Public Kroki is not a
  sensible default; it requires explicit user approval.
- **Report the path in Step 8.** State which backend rendered and whether the
  source left the machine — e.g. *"rendered via public Kroki (source uploaded to
  kroki.io)"* vs *"rendered via host PlantUML CLI (stayed on your machine)."*
  The user should never have to guess whether their diagram crossed the network.
