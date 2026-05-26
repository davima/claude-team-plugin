# Team Plugin for Claude Code

Five sparring-partner slash commands for the build cycle: PM, Principal Engineer, QA, Coder, Code Reviewer.

Each command is meant to be run in its own Claude Code session (separate terminal tab) so you can actually debate with it — Claude Code subagents can't dialogue, but separate sessions can. The commands lean on existing Claude Code skills (`grill-me`, `grill-with-docs`, `prd-review`, `superpowers:test-driven-development`, etc.) rather than reinventing them.

## Commands

| Command | Role | Primary skill |
|---|---|---|
| `/team:pm` | Product Critic — grills requirements, missing flows, edge cases | `prd-review`, `grill-me` |
| `/team:principal-engineer` | Principal Engineer — grills tech design, updates ADRs/CONTEXT.md inline | `grill-with-docs` |
| `/team:qa` | Test Strategist — produces a test plan with every scenario tagged `[unit]`/`[integration]`/`[e2e]` | `grill-me` |
| `/team:coder` | TDD implementer — worktree-isolated, writes Playwright specs for `[e2e]` scenarios, browser-validates UI | `superpowers:test-driven-development`, `superpowers:executing-plans`, `superpowers:using-git-worktrees` |
| `/team:code-reviewer` | Fresh-eyes reviewer — uses built-in `/review` + `simplify`, walks UI changes via `chrome-devtools-mcp` | `simplify` |

## Project-specific context

Each command checks for `.claude/personas/<role>.md` in the current working directory when invoked. If the file exists, its content is read and treated as **project-specific augmentation** before running the generic persona — typically:

- Which docs to load up-front
- Scope reminders ("on this project, the PM also covers UX")
- Output paths
- Project-specific invariants to enforce

If the file doesn't exist, the command runs the generic persona alone.

### Example `your-project/.claude/personas/pm.md`

```markdown
# Your Project — PM context

## Load these docs first via Read tool

- `docs/prd.md`
- `docs/design-doc.md`

## Outputs

Write notes to `docs/pm-critique.md`.
```

That's it — no frontmatter, no slash-command magic. Plain markdown that the plugin command reads as instructions.

## Installation

### First time on a new machine

```bash
git clone https://github.com/davima/claude-team-plugin.git ~/projects/claude-team-plugin
```

Then in Claude Code:

```
/plugin marketplace add ~/projects/claude-team-plugin
/plugin install team@personal
/reload-plugins
```

Verify with `/team:` autocomplete — all five commands should appear.

### Updating

```bash
cd ~/projects/claude-team-plugin && git pull
```

Then in Claude Code: `/reload-plugins`. No reinstall needed.

### Uninstalling

In Claude Code: `/plugin uninstall team@personal`. To also remove the marketplace: `/plugin marketplace remove personal`.

## Customization

The generic personas live in `plugins/team/commands/*.md`. Edit those files to tune the role definitions — what to grill on, which skills to invoke, what to refuse. Changes are picked up automatically; you only need to `/reload-plugins` if you change the plugin manifest or add/remove command files.

Project-specific tweaks belong in `your-project/.claude/personas/<role>.md`, not in this plugin. That keeps the plugin reusable across projects.

## Repository layout

```
.
├── .claude-plugin/
│   └── marketplace.json            ← marketplace named "personal"
├── plugins/
│   └── team/
│       ├── .claude-plugin/
│       │   └── plugin.json         ← plugin named "team"
│       └── commands/
│           ├── pm.md               → /team:pm
│           ├── principal-engineer.md → /team:principal-engineer
│           ├── qa.md               → /team:qa
│           ├── coder.md            → /team:coder
│           └── code-reviewer.md    → /team:code-reviewer
└── README.md
```
