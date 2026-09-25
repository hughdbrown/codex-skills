# Codex Skills

A small collection of reusable skills for Codex. Each skill is a directory with a `SKILL.md` file that tells Codex when and how to carry out a repeatable workflow. The skills here focus on software development tasks.

## Skills

| Skill | What it does |
| --- | --- |
| [`codex-team`](codex-team/) | Guides a Sol technical lead and Luna workers through planning, bounded implementation, review, and integration. The source prompt and supporting reference are included alongside the skill. |
| [`roborev-pull-request-reviewer`](roborev-pull-request-reviewer/) | Reviews a pull request or current branch for likely issues and produces a report with findings and verification results. |

## Use a skill

To make a skill available to Codex, copy its directory into your personal skills folder or a repository's `.agents/skills/` directory:

```sh
# Personal installation
mkdir -p ~/.agents/skills
cp -R codex-team ~/.agents/skills/

# Or install it for a single repository, from that repository's root
mkdir -p .agents/skills
cp -R /path/to/codex-skills/roborev-pull-request-reviewer .agents/skills/
```

After installation, ask Codex to use the skill by name, or describe a task that matches its `SKILL.md` description. For example:

```text
Use the roborev:pull-request-reviewer skill to review the current branch.
```

## Repository layout

```text
codex-team/
  SKILL.md       Skill instructions
  PROMPT.md      Source workflow prompt
  REFERENCE.md   Supporting reference
roborev-pull-request-reviewer/
  SKILL.md       Skill instructions
```

## Add a skill

Create a directory for the skill and add a `SKILL.md` with YAML front matter containing a `name` and a concise `description`, followed by the workflow instructions. Put longer references, templates, or scripts in the same directory and link to them from `SKILL.md`.
