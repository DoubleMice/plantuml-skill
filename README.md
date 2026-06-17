# plantuml-skill

A public, local-first PlantUML skill for AI coding agents.

This skill helps Codex, Claude Code, OpenCode, and compatible local agents
create PlantUML diagrams, save `.puml` source files, and render PNG/SVG output
when host PlantUML dependencies are already installed. It is designed for
security-sensitive engineering work: local rendering is the default, and public
Kroki rendering is allowed only after explicit user approval.

## Quick Install

Copy the prompt below into your local agent. The prompt is intentionally
agent-neutral: the agent should detect its own runtime and install this public
repo into the right local skill directory.

One-line version:

```text
Install the public PlantUML Agent Skill from https://github.com/DoubleMice/plantuml-skill into the correct local skills directory for the agent you are running in; detect Codex, Claude Code, or OpenCode automatically; do not overwrite an existing plantuml-skill directory; verify SKILL.md is installed at the target root; report the final path and whether a restart/reload is needed.
```

Expanded version:

```text
Install the public PlantUML Agent Skill from https://github.com/DoubleMice/plantuml-skill into the correct local skills directory for the agent you are running in.

Detect the current agent/runtime:
- If you are Codex, install repo path "." as skill name "plantuml-skill" into ${CODEX_HOME:-$HOME/.codex}/skills/plantuml-skill. Prefer the built-in Codex skill installer at ${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-installer/scripts/install-skill-from-github.py when available.
- If you are Claude Code, install into ~/.claude/skills/plantuml-skill by default. If I explicitly ask for project-local install, use .claude/skills/plantuml-skill instead.
- If you are OpenCode, install into ~/.config/opencode/skills/plantuml-skill by default. If I explicitly ask for project-local install, use .opencode/skills/plantuml-skill instead.
- If you cannot confidently detect the current agent/runtime, ask me which target to use before making changes.

Use the public GitHub repo directly. You may use gh, git, the Codex skill installer, or another downloader available in the current environment. The installed directory must contain SKILL.md directly at the target directory root, not nested under an archive wrapper directory.

Do not overwrite an existing plantuml-skill directory. If it already exists, stop and tell me. After installation, verify that SKILL.md exists at the installed path, run any available skill validation for the current agent, report the final path, and tell me whether the current agent needs a restart or reload to discover the skill.
```

OpenCode can also discover Claude-compatible skill installs under
`~/.claude/skills`, so one Claude Code install can be shared by both tools when
that is the desired setup. OpenCode also supports agent-compatible installs
under `~/.agents/skills` and `.agents/skills`.

## What It Does

- Generates PlantUML source for sequence, component, class, ER, activity, state,
  use-case, C4, mind-map, and Gantt diagrams.
- Renders to PNG or SVG with host-installed `plantuml` and Graphviz (`dot`).
- Falls back to a host `plantuml.jar` when Java, Graphviz, and the jar are
  available.
- Extracts and renders PlantUML blocks embedded in Markdown.
- Builds diagrams from existing source code by reading real code structure
  instead of inventing architecture.
- Reports which renderer was used and whether the diagram source stayed local.

## Agent Support

| Agent | Global install path | Project-local install path | Invocation |
|---|---|---|---|
| Codex | `${CODEX_HOME:-$HOME/.codex}/skills/plantuml-skill` | N/A | Automatic skill selection |
| Claude Code | `~/.claude/skills/plantuml-skill` | `.claude/skills/plantuml-skill` | Automatic or `/plantuml-skill` |
| OpenCode | `~/.config/opencode/skills/plantuml-skill` | `.opencode/skills/plantuml-skill` | Via OpenCode's native `skill` tool |

The same `SKILL.md` is used for all three agents. Its frontmatter intentionally
keeps only `name` and `description` for compatibility; source, dependency, and
origin details live in this README and in the skill body.

Official references:

- Claude Code skills: https://code.claude.com/docs/en/skills
- OpenCode agent skills: https://opencode.ai/docs/skills/

## Dependencies

Installing this skill only copies instruction files. It does not install Java,
Graphviz, `plantuml`, or `plantuml.jar`.

Dependency by task:

| Task | Required dependencies |
|---|---|
| Generate `.puml` source | None |
| Render with host PlantUML CLI | `plantuml` and Graphviz (`dot`) |
| Render with host PlantUML jar | Java, Graphviz (`dot`), and `plantuml.jar` |
| Render with public Kroki | `curl` plus explicit user approval |

Install host rendering dependencies:

```bash
# macOS
brew install plantuml graphviz

# Ubuntu / Debian
sudo apt-get update
sudo apt-get install -y plantuml graphviz
```

Verify host rendering dependencies:

```bash
plantuml -version
dot -V
```

Render locally:

```bash
plantuml -tpng diagram.puml
plantuml -tsvg diagram.puml
```

Manual jar fallback:

```bash
PLANTUML_JAR=/path/to/plantuml.jar
java -jar "$PLANTUML_JAR" -tpng diagram.puml
```

## Rendering Policy

Renderer precedence:

1. Use an explicitly requested host renderer when the user names one.
2. Prefer host `plantuml`.
3. Use host `plantuml.jar` only when Java, Graphviz, and the jar are available.
4. If no host renderer is available, generate `.puml` only and report that
   rendering was skipped.
5. Use public Kroki only when the user explicitly approves uploading the
   diagram source for the current task.

The skill should never silently upload diagram source to a public service.

## Generated Diagram

The PlantUML source for this overview is tracked at
[`assets/plantuml-skill-flow.puml`](assets/plantuml-skill-flow.puml).

![PlantUML skill flow](assets/plantuml-skill-flow.svg)

## Origin

This repository is derived from the upstream skill:

- Original repository: https://github.com/Agents365-ai/plantuml-skill
- Original skill path: `skills/plantuml-skill`
- Upstream licensing note: the original skill metadata declares MIT licensing.

This public fork keeps the original PlantUML workflow while changing the
operational default: render locally with host dependencies, avoid Docker as a
runtime requirement, and keep network rendering behind explicit approval.

## Validation

Validate the skill metadata when the Codex skill creator tools are available:

```bash
python3 "${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-creator/scripts/quick_validate.py" \
  "${CODEX_HOME:-$HOME/.codex}/skills/plantuml-skill"
```

After installing or updating the skill, restart or reload the agent if its skill
discovery does not pick up changes live.
