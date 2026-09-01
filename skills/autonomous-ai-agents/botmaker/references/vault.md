# Vault (`Bots/`)

Botmaker owns `Bots/` in the fleet vault. This layer assumes an Obsidian vault because that is what we run; any notes system with frontmatter and links works — adapt the conventions, keep the rule: **nothing gets documented before certification.** A nightly tidy job may own `Sessions/`; do not steal that cron. Do not own `Servers/` or `Services/` except links.

Vault path:

`<your-vault-path>`

Quote it in shell if it contains spaces (cloud-synced vault paths often do). File tools take the absolute path.

Load your notes skill (if you have one) for filesystem conventions. This file is the *bot* layer on top.

## When to write

**After certification.** Seeding a profile is not a ship. Uncertified bots do not get `Bots/<name>.md` or a roster row.

Exception: `@botmaker` itself was scaffolded in the same session as this skill; its note may exist as a specialist reference marked **certification pending** until the first Bot Chat dogfoods the process. Do not copy that exception onto children.

## Notes

| Note | Frontmatter | Role |
|---|---|---|
| `Bots/<name>.md` | `type: bot-reference`, `created`, `updated`, `profile`, `tags` | Specialist reference after certification. Not a SOUL dump. Training loop + runbook pointer + independence pin + pitfalls. |
| `Bots/making-bots.md` | `type: living-guide` | Living process. Human twin of this skill. Roster table at top — add a row when a bot **ships**. Changelog at bottom. |
| `Home.md` | `type: vault-root` | One index line under **Bots**. Never append running logs. |
| `Sessions/YYYY-MM-DD-<slug>.md` | `type: session` | What we did this session. Wikilink the bot note + `making-bots`. One Index line, newest first. |

Match your vault's existing tone: YAML frontmatter, wiki-style links, short, no credentials. Once your first specialist is certified, its note becomes the worked example — steal structure, not facts.

## Roster

`making-bots.md` table:

```markdown
| `@name` | `name` | <model> / <provider> | one-line job |
```

`Home.md` **Bots** section: `[[name]] — \`@name\`, one-line job`.

## Lockstep with this skill

When the process changes (symlink rule, create leftover `base_url`, human gate, …), patch **both** this skill (`~/.hermes/skills/autonomous-ai-agents/botmaker/`) **and** `Bots/making-bots.md` in the same turn. Profile files are symlinks; write the canonical tree.

Ops source of truth for a *child* bot is that child's skill, not the vault note. The vault note points at the skill.
