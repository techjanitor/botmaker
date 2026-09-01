# botmaker

A [Hermes Agent](https://github.com/NousResearch/hermes-agent) skill + SOUL for a bot whose only job is minting other specialist bots.

The loop it enforces:

**interview → one-screen SOUL draft → human sign-off → scaffold (`--no-skills`, pinned brain, file-level skill links) → certification in the child's own chat → documentation in the vault**

> Seeding files is not a bot.

## Why

A SOUL.md dump on a cloned profile is a costume. A specialist has one job, a brain pinned off its own failure path, earned constraints, and a memory it wrote itself. Botmaker exists to make the second kind and refuse the first.

## What Hermes is

[Hermes Agent](https://github.com/NousResearch/hermes-agent) is Nous Research's open-source agent framework: profiles (bots), skills, SOULs, persistent memory, bot-to-bot messaging. Docs: <https://hermes-agent.nousresearch.com>. This repo is a skill and a SOUL *for* Hermes, not a fork of it.

## What's in the repo

```
profile/SOUL.md                          the botmaker SOUL — fill the <placeholders>, install per below
profile/config.yaml                      example profile config — the headless-provisioning security flag, explained
profile/profile.yaml                     example roster metadata — the one-liner other bots read, plus the UI title
skills/autonomous-ai-agents/botmaker/
├── SKILL.md                             the runbook: interview, scaffold, certification, failure-mode index
├── references/soul-craft.md             what goes in a SOUL, and the human gate
├── references/process.md                independence pin, --no-skills, shared-skill mechanics, kanban intake
├── references/vault.md                  documenting the fleet (after certification, never before)
├── scripts/link_skill_tree.py           file-level symlink linker for shared skills
└── scripts/drift_check.py               doc-drift tripwire — run after every mint
```

The `skills/` subtree mirrors the canonical Hermes tree, so every path referenced inside the docs is true after install.

## Prerequisites

- Hermes CLI with a working default profile (`hermes profile create --help` succeeds)
- **At least two model providers configured.** The independence pin is the heart of the method: a bot that operates a service must not run on that service's inference path. If your fleet runs local inference, that means a hosted provider for the operators.
- A documentation home for the fleet — any markdown directory works (Obsidian conventions shown in `references/vault.md`)
- Optional: the [Honcho](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory-providers) memory provider

## Install

Installing botmaker is your first supervised run of its own method.

1. **Alias preflight** for the name `botmaker` (or whatever you call yours) — the two checks from `SKILL.md`:

   ```sh
   zsh -lc 'which -a botmaker'
   test -e "$HOME/.local/bin/botmaker" || test -L "$HOME/.local/bin/botmaker"
   ```

   If either finds anything, create with `--no-alias`. (Why this matters: Hermes writes an alias wrapper into `~/.local/bin`, `Path.write_text()` follows symlinks, and that combination has overwritten a real installed CLI binary. The skill tells the whole story.)

2. **Copy the canonical skill tree** into the default Hermes home:

   ```sh
   cp -R skills/autonomous-ai-agents/botmaker ~/.hermes/skills/autonomous-ai-agents/botmaker
   ```

3. **Create the profile:**

   ```sh
   hermes profile create botmaker --no-skills --description "Designs, scaffolds, certifies, and documents specialist Hermes bots."
   ```

4. **Pin the brain** and clear any model block inherited from your default profile:

   ```sh
   hermes -p botmaker config set model.provider <provider>
   hermes -p botmaker config set model.default <model>
   hermes -p botmaker config unset model.base_url
   hermes -p botmaker config unset model.api_key
   hermes -p botmaker config get model   # must print only default + provider
   ```

5. **Install the SOUL.** Fill the `<placeholders>` in `profile/SOUL.md` (table below), then write it to `~/.hermes/profiles/botmaker/SOUL.md`. If you do this through the bot's own tools, Hermes will prompt you to approve the write — that prompt is the human gate working. Approve it yourself; don't disable it for interactive use. (Provisioning bots headless from kanban cards is the one scenario that needs the gate's tooling half off — see `profile/config.yaml` and the security note below.)

6. **Link the skill into the profile with its own linker** (file-level symlinks — a directory symlink is invisible to Hermes's skill discovery on Python 3.11):

   ```sh
   python3 ~/.hermes/skills/autonomous-ai-agents/botmaker/scripts/link_skill_tree.py \
     ~/.hermes/skills/autonomous-ai-agents/botmaker \
     ~/.hermes/profiles/botmaker/skills/autonomous-ai-agents/botmaker
   ```

7. **Smoke test.** Open the bot's chat and ask it to mint something deliberately fuzzy ("a bot to help with my projects"). It should refuse to scaffold and ask one sharp question. Give it a sharp job; it should stop at the sign-off gate with a one-screen draft.

8. **Drift tripwire.** `python3 ~/.hermes/skills/autonomous-ai-agents/botmaker/scripts/drift_check.py` should exit clean; run it after every mint and every doc patch. Point `BOTMAKER_VAULT_GUIDE` at your vault's `Bots/making-bots.md` to enable the roster-vs-profiles check. The design it enforces: every rule has one home — method in the skill tree, fleet state in the vault roster, history in the changelog.

## What to customize

| Placeholder | Where | Meaning |
|---|---|---|
| `<model>` / `<provider>` | `profile/SOUL.md`, throughout the skill | The independence pin. Choose a provider that survives the failure your bots must outlive. |
| `<your-vault-path>` | `references/vault.md`, SKILL.md prerequisites | Your fleet's documentation home. Any markdown directory works; Obsidian conventions are shown. |
| `<peer>` / `<peer-url>` / `<keyfile>` | "site-specific" blocks in SKILL.md and process.md | Our external peer bridge, shown as a worked example. Adapt if your fleet has bot-to-bot peers outside Hermes; delete if not. |
| `<review-skill>` | process.md | Placeholder for any skill your automation injects with `--skills`. |
| Honcho sections | SKILL.md, process.md | Only if you use the Honcho memory provider; delete otherwise. |
| `security.protected_instruction_files` | `profile/config.yaml` | `false` is what lets botmaker provision bots unattended from kanban cards; leave the default `true` if you run it interactively. See the security note below. |
| `author:` frontmatter | SKILL.md | Make it yours once you've adapted the skill. |

Other angle-bracket tokens (`<category>`, `<name>`, `<bot>`, `NAME`) are per-command values, the same convention the Hermes docs use.

## Field notes

The pitfalls list in `SKILL.md` is earned, not hypothetical. Three of the scars, in brief:

- **The alias clobber.** A profile that shared its name with an installed CLI overwrote the real 107MB binary through a `~/.local/bin` symlink. The preflight exists because the obvious check (`which`) is exactly the one that fails.
- **The create leftover.** `profile create` copies the default profile's model block, so the freshly "independent" bot was silently still talking to the local GPU box. Hence the unset-and-verify step.
- **The costume.** An operator bot shipped as "You are Grok" in a hat and had to be rewritten down to its job. Identity is the job name; voice inheritance is tone, not self.

## Security note

Hermes protects agent-instruction files: writes to a profile's `SOUL.md` prompt the human by default (`security.protected_instruction_files: true`). That default is correct — the prompt is this method's human gate materialized in tooling.

`profile/config.yaml` shows the one deliberate exception: botmaker's own profile runs with the flag set to `false`, because that is what allows it to provision bots from kanban cards unattended — a headless worker has no approval surface, so with the flag on, the child's `SOUL.md` write blocks on a prompt nobody can see, times out, and ships a stub. With the flag off, the signed spec on the kanban card becomes the only gate: no signature, no write. If you run botmaker interactively instead, leave the default alone — and never set the flag fleet-wide.

## License

MIT — see [LICENSE](LICENSE).
