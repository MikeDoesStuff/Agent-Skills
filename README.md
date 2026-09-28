# Agent-Skills

A collection of skills for AI coding agents. Each skill is a self-contained folder with a `SKILL.md` file that an agent loads on demand to guide its behavior for a specific task.

## Repository layout

```
skills/
  <skill-name>/
    SKILL.md        # frontmatter (name, description) + instructions
    ...             # optional supporting files
```

## Skills

| Skill | Description |
| --- | --- |
| [jg-code-search](skills/jg-code-search/SKILL.md) | Guidance for using the preinstalled `jg` (jevgrep) CLI to search local code repositories semantically by behavior or intent, and exactly for known identifiers. |

## Using these skills

### Manual install

Copy the skill folder into your agent's skills directory:

```powershell
# OpenCode / shared agents dir
New-Item -ItemType Directory -Force "$HOME\.agents\skills" | Out-Null
Copy-Item -Recurse skills\jg-code-search "$HOME\.agents\skills\"

# Codex
New-Item -ItemType Directory -Force "$HOME\.codex\skills" | Out-Null
Copy-Item -Recurse skills\jg-code-search "$HOME\.codex\skills\"
```

Or on bash/zsh:

```bash
mkdir -p ~/.agents/skills && cp -r skills/jg-code-search ~/.agents/skills/
mkdir -p ~/.codex/skills && cp -r skills/jg-code-search ~/.codex/skills/
```

### Install via agent

Agents that support `npx skills` can install directly:

```bash
npx skills add github.com/MikeDoesStuff/Agent-Skills --skill jg-code-search
```

## Requirements

- Skills here assume their runtime dependencies are already installed and configured on the machine (for example, `jg-code-search` requires the `jg` CLI). See each skill's `SKILL.md` for specifics.

## License

[MIT](LICENSE) © Michael Hughes
