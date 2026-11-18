# Lisper

Lisper is an Agent Skill for strict Common Lisp and Emacs Lisp engineering work. Modeled after [Rustacean](https://github.com/RyanKung/Rustacean), it guides AI coding agents to enforce warning-free compilation (`byte-compile-error-on-warn` / SBCL `STYLE-WARNING`), `lexical-binding: t`, hygienic macros, structured conditions (`define-condition` / `define-error`), pure functional cores, explicit cons-cell ownership, `-p` semantic predicates, and `checkdoc` / `package-lint` / `sblint` discipline.

The repository follows the common `SKILL.md` package shape:

```text
Lisper/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── README.md
```

Use symlinks during development so every runtime sees the same working copy.

## Antigravity

Antigravity discovers global skills from `~/.gemini/config/skills/<skill-name>/SKILL.md` and workspace skills from `.agents/skills/<skill-name>/SKILL.md`.

Install globally for Antigravity:

```bash
cd /path/to/Lisper
mkdir -p ~/.gemini/config/skills
ln -s "$(pwd)" ~/.gemini/config/skills/lisper
```

## Claude Code

Claude Code discovers personal skills from `~/.claude/skills/<skill-name>/SKILL.md` and project skills from `.claude/skills/<skill-name>/SKILL.md`.

Install for all Claude Code projects:

```bash
cd /path/to/Lisper
mkdir -p ~/.claude/skills
ln -s "$(pwd)" ~/.claude/skills/lisper
```

Install for one project:

```bash
cd /path/to/Lisper
mkdir -p /path/to/repo/.claude/skills
ln -s "$(pwd)" /path/to/repo/.claude/skills/lisper
```

Invoke explicitly in Claude Code:

```text
/lisper
```

## Codex

Codex discovers user-level skills from `~/.agents/skills` and repository-scoped skills from `.agents/skills`.

Install for all Codex projects:

```bash
cd /path/to/Lisper
mkdir -p ~/.agents/skills
ln -s "$(pwd)" ~/.agents/skills/lisper
```

Install for one repository:

```bash
cd /path/to/Lisper
mkdir -p /path/to/repo/.agents/skills
ln -s "$(pwd)" /path/to/repo/.agents/skills/lisper
```

Invoke explicitly in Codex:

```text
$lisper
```

## OpenClaw

Install in the shared Agent Skills path:

```bash
cd /path/to/Lisper
mkdir -p ~/.agents/skills
ln -s "$(pwd)" ~/.agents/skills/lisper
```

Install for one OpenClaw workspace:

```bash
cd /path/to/Lisper
mkdir -p /path/to/openclaw-workspace/.agents/skills
ln -s "$(pwd)" /path/to/openclaw-workspace/.agents/skills/lisper
```

## Hermes

Install directly into the active Hermes skills directory:

```bash
cd /path/to/Lisper
mkdir -p ~/.hermes/skills
ln -s "$(pwd)" ~/.hermes/skills/lisper
```

Invoke from the CLI:

```bash
hermes --skills lisper
```

## Verify

Check that the installed path points to this repository:

```bash
ls -l ~/.gemini/config/skills/lisper
ls -l ~/.claude/skills/lisper
ls -l ~/.agents/skills/lisper
ls -l ~/.hermes/skills/lisper
```

The skill root must contain:

```text
SKILL.md
agents/openai.yaml
README.md
```

## Uninstall

Remove the installed symlink or copied directory for each runtime:

```bash
rm -f ~/.gemini/config/skills/lisper
rm -f ~/.claude/skills/lisper
rm -f ~/.agents/skills/lisper
rm -f ~/.openclaw/skills/lisper
rm -f ~/.hermes/skills/lisper
```
