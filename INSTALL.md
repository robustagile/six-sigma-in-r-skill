# Installing the Six Sigma in R skill

This skill ships as a folder: `skills/six-sigma-r/`. Installing it means making
that folder discoverable by your coding agent.

There are four common setups, crossed on two axes:

|                | **Global** (every project)            | **Local** (one project)                |
| -------------- | ------------------------------------- | -------------------------------------- |
| **Claude Code** | `~/.claude/skills/six-sigma-r/`       | `<project>/.claude/skills/six-sigma-r/` |
| **Codex**       | referenced from `~/.codex/AGENTS.md`  | referenced from `<project>/AGENTS.md`   |

Pick the row (agent) and column (scope) that matches your situation. Steps are
written for Linux / macOS / WSL; Windows PowerShell equivalents are noted where
they differ.

---

## Claude Code

Claude Code auto-discovers skills in two directories:

- `~/.claude/skills/` — **global**, available in every session.
- `<project>/.claude/skills/` — **local**, available only when Claude Code is
  launched from inside that project.

Each skill lives in its own sub-folder with a `SKILL.md` at the top.

### Option 1 — global for Claude Code

```bash
# from the repository root
mkdir -p ~/.claude/skills
cp -r skills/six-sigma-r ~/.claude/skills/
```

To keep it updated by `git pull`, symlink instead of copy:

```bash
mkdir -p ~/.claude/skills
ln -s "$(pwd)/skills/six-sigma-r" ~/.claude/skills/six-sigma-r
```

Windows PowerShell (symlink requires Developer Mode or an admin shell):

```powershell
New-Item -ItemType Directory -Force -Path $HOME\.claude\skills | Out-Null
New-Item -ItemType SymbolicLink -Path $HOME\.claude\skills\six-sigma-r `
         -Target (Resolve-Path .\skills\six-sigma-r)
```

### Option 2 — local (per-project) for Claude Code

Inside the project where you want the skill available, from that project's
root:

```bash
mkdir -p .claude/skills
cp -r /path/to/sixsigmainr/skills/six-sigma-r .claude/skills/
# or, to track the source repo:
ln -s /path/to/sixsigmainr/skills/six-sigma-r .claude/skills/six-sigma-r
```

Commit `.claude/skills/six-sigma-r/` (or the symlink) if you want teammates on
the same project to pick it up automatically.

### Verify

Start Claude Code and ask something like *"write an R P chart from this CSV"*.
The skill should trigger — you'll see `six-sigma-r` referenced in the
available-skills list, and the response should use the skill's conventions
(base R by default, proper control-chart formulas).

To confirm discovery without generating output, start a session and type:

```
/skills
```

and look for `six-sigma-r` in the list.

---

## Codex (OpenAI `codex` CLI)

Codex doesn't have an auto-discovered `skills/` directory the way Claude Code
does. It uses `AGENTS.md` files as persistent instructions:

- `~/.codex/AGENTS.md` — **global**, loaded for every Codex session.
- `<project>/AGENTS.md` — **local**, loaded when Codex runs in that project.

The installation pattern is: keep the skill folder somewhere on disk (either
checked out with this repo or copied under the relevant `.codex` / project
folder), and have `AGENTS.md` point Codex at `SKILL.md`.

### Option 3 — global for Codex

Pick a permanent location for the skill folder. The cleanest is
`~/.codex/skills/`, mirroring the Claude Code layout:

```bash
mkdir -p ~/.codex/skills
cp -r skills/six-sigma-r ~/.codex/skills/
# or symlink:
ln -s "$(pwd)/skills/six-sigma-r" ~/.codex/skills/six-sigma-r
```

Then append a pointer block to the global `AGENTS.md` so Codex loads it:

```bash
mkdir -p ~/.codex
cat >> ~/.codex/AGENTS.md <<'EOF'

## Skill: six-sigma-r

When the user asks for R code related to Six Sigma, statistical process
control, control charts, process capability, DPMO / sigma level, hypothesis
tests, ANOVA, sample size / power analysis, Pareto charts, or any related
quality-engineering task, follow the instructions in
`~/.codex/skills/six-sigma-r/SKILL.md` and consult the per-technique files
under `~/.codex/skills/six-sigma-r/references/` as directed by SKILL.md.
EOF
```

(If your Codex version supports file-include directives in `AGENTS.md`, you
can replace the pointer block with the include form supported by your
version instead of the prose description above. The pointer-by-prose form
works with any Codex release that reads `AGENTS.md`.)

### Option 4 — local (per-project) for Codex

Inside your project root:

```bash
mkdir -p .codex-skills
cp -r /path/to/sixsigmainr/skills/six-sigma-r .codex-skills/
# or symlink:
ln -s /path/to/sixsigmainr/skills/six-sigma-r .codex-skills/six-sigma-r
```

Then add a pointer to the project's `AGENTS.md` (create it if absent):

```markdown
## Skill: six-sigma-r

When the user asks for R code related to Six Sigma, SPC, control charts,
capability, DPMO / sigma level, hypothesis tests, ANOVA, sample size / power
analysis, or Pareto / run / histogram / scatter charts, follow the instructions
in `.codex-skills/six-sigma-r/SKILL.md` and consult the per-technique files
under `.codex-skills/six-sigma-r/references/`.
```

Commit `.codex-skills/` and the `AGENTS.md` update so everyone on the project
gets the skill.

### Verify (Codex)

Start Codex in the project (or any project, for the global install) and ask:

```
Write me an R script that computes Cpk and Ppk for a vector of measurements.
```

The response should follow the skill's conventions (base R, both Cp and Pp
families, explicit short-term vs long-term σ discussion). If Codex doesn't
pick it up, check that the `AGENTS.md` pointer path is correct and that the
skill folder actually exists at that path.

---

## Uninstalling

Remove whichever copies / symlinks you created:

```bash
# Claude Code
rm -rf ~/.claude/skills/six-sigma-r
rm -rf <project>/.claude/skills/six-sigma-r

# Codex
rm -rf ~/.codex/skills/six-sigma-r
rm -rf <project>/.codex-skills/six-sigma-r
# and delete the "## Skill: six-sigma-r" block from the relevant AGENTS.md
```

## Troubleshooting

- **Skill not triggering in Claude Code** — confirm the folder is at
  `~/.claude/skills/six-sigma-r/` (global) or `<project>/.claude/skills/six-sigma-r/`
  (local) and that `SKILL.md` sits at that folder's top level. Restart the
  session after install.
- **Skill triggers but reference files aren't read** — the references are
  loaded on demand by path reference from `SKILL.md`. Make sure
  `references/` is a sibling of `SKILL.md` inside `six-sigma-r/`, not moved
  elsewhere.
- **Codex ignores the AGENTS.md pointer** — check that the Codex version in
  use actually reads `~/.codex/AGENTS.md` (run `codex --help` or consult its
  docs). Older or alternative builds may use a different config path.
- **R / ggplot2 isn't found when the generated script runs** — that's a
  runtime issue, not a skill issue. Install R and any packages the script
  requires (see `README.md` → Prerequisites).
