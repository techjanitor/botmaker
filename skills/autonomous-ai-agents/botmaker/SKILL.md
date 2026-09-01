---
name: botmaker
description: "Use when creating specialist Hermes bots."
version: 0.1.0
author: techjanitor
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [hermes, bots, profiles, soul]
    category: autonomous-ai-agents
    related_skills: [hermes-agent]
---

# Botmaker

Design, scaffold, certify, and document specialist Hermes bots. Load this skill on every in-scope question. You draft; the child earns. Seeding files is not a bot.

Details: `references/soul-craft.md`, `references/process.md`, `references/vault.md`. Shared-skill linker: `scripts/link_skill_tree.py`.

## When to Use

- Your human wants a new specialist bot, or a SOUL drafted for one
- An existing specialist bot's identity/job/voice is wrong and needs a SOUL rewrite (human gate)
- `Bots/` in the vault needs a specialist note, roster row, or `making-bots.md` changelog
- A shared skill is about to be copied into a profile (stop; symlink files instead)

Don't use for: operating the inference server, ComfyUI, or any service; general coding; rewriting default `~/.hermes/SOUL.md`; spawning `botmaker-2`. `@hermes` is not the bot coordinator — you are.

## Prerequisites

- Hermes CLI (`hermes profile create --help`)
- Canonical skills live under `~/.hermes/skills/` (default Hermes home), **not** `$HERMES_HOME/skills` when you are the botmaker profile
- Vault: `<your-vault-path>` (see `references/vault.md`)
- Living process: vault `Bots/making-bots.md` (human twin of this skill; create it on first ship). Patch both when the process changes.

## How to Run

1. Interview. Refuse to scaffold until job / independence failure / not-list are sharp. See Procedure.
2. Draft a one-screen SOUL (`references/soul-craft.md`). Stop. Your human signs it.
3. Scaffold (`references/process.md`). **Alias preflight first.** `--no-skills`, pin the brain, file-level skill links, USER.md lockstep sentence if the skill is shared.
4. Kickoff in the **child's** Bot Chat. Do not babysit its tools. Do not write its `MEMORY.md`.
5. After certification: vault note + roster row (`references/vault.md`). Not before.

## Quick Reference

| Need | Do |
|---|---|
| New bot | Alias preflight (below) → Interview → SOUL draft → human sign-off → `hermes profile create NAME --no-skills --description "…"` |
| Pin a brain | `hermes -p NAME config set model.provider …` then `model.default …`. Then **unset** `model.base_url` and `model.api_key` if create copied your default profile's local `base_url` |
| Shared skill | Canonical `~/.hermes/skills/<cat>/<name>/`; profile files via `scripts/link_skill_tree.py`. Never a directory symlink. Load linked `references/` with `read_file` on that canonical path — `skill_view(..., file_path=…)` escapes |
| Discovery bug | Python 3.11 `Path.rglob("SKILL.md")` skips directory symlinks → empty index |
| Vault after cert | `Bots/<name>.md` (`type: bot-reference`), roster row on `making-bots.md`, one line on `Home.md` |
| Child memory | Child writes it. You do not. |

## Procedure

### A. Interview (before any `profile create`)

Ask, and do not proceed until 1–3 are sharp:

1. One-sentence job. If "and also," split into two bots. Job may be a CLI, an HTTP API, a GUI sock — ComfyUI counts. "CLI-only" is how we *create* the profile, not what the specialist is allowed to operate.
2. Failure the brain must survive → model pin (if the bot's job lives on a local box, that usually means a hosted provider — the bot must still think when the box is down).
3. What it is not.
4. Does default Hermes need the same skill? → canonical under `~/.hermes/skills/` + file-level symlinks.
5. Who is the client to ping (`@hermes`, another specialist, nobody).
6. Voice: inherit vs write. Inheritance is tone, not the model's stock identity — that is costume unless the bot *is* the persona bot.

### B. SOUL draft

Write one screen. Identity (job name first), Job, Hard constraints as their own heading, Voice, What you are not, Disagreement. No live pins, no IPs, no last-incident notes, no "helpful assistant," no pasted stock-model `# Style` block. Full rules in `references/soul-craft.md`.

**Human gate:** do not run `hermes profile create` until your human signs the draft. Iterate in chat, not on disk.

### C. Scaffold

**Alias preflight — mandatory, before any `profile create NAME`.** Hermes writes `~/.local/bin/<name>` (`exec hermes -p <name>`). `Path.write_text()` follows symlinks, so a wrapper can overwrite a real CLI in place (war story: a profile that shared its name with an installed CLI overwrote the real 107MB binary living at the far end of a `~/.local/bin` symlink — the alias wrapper was written *through* the link, straight over the binary). Built-in `which` collision check is useless under launchd PATH (`/usr/bin:/bin:/usr/sbin:/sbin`) — no `~/.local/bin`. Do not trust this process's `which` either.

```text
zsh -lc 'which -a NAME'
# and, even if which is empty:
test -e "$HOME/.local/bin/NAME" || test -L "$HOME/.local/bin/NAME"
```

If **either** finds anything: do **not** let Hermes install the default alias. Create with `--no-alias`, or `hermes profile alias NAME --name CUSTOM` only after the **same two checks** pass for `CUSTOM`. Extra caution if `~/.local/bin/NAME` is a **symlink** — that is the binary-destroying case.

Common landmine: Claude Code installs `claude` as a symlink under `~/.local/bin` — a bot named `claude` clobbers it exactly this way. So does any CLI whose entry point lives in, or resolves through, `~/.local/bin`. Hermes only reserves `hermes`, `default`, `test`, `tmp`.

If a CLI later acts like Hermes (`exec hermes -p` in a tiny file): delete the impostor, reinstall the real tool, rename or drop the Hermes alias.

```text
HERMES_HOME="$HOME/.hermes" hermes profile create NAME --no-skills --description "one or two sentences"
HERMES_HOME="$HOME/.hermes" hermes -p NAME config set model.provider <provider>
HERMES_HOME="$HOME/.hermes" hermes -p NAME config set model.default <model>
HERMES_HOME="$HOME/.hermes" hermes -p NAME config unset model.base_url
HERMES_HOME="$HOME/.hermes" hermes -p NAME config unset model.api_key
HERMES_HOME="$HOME/.hermes" hermes profile describe NAME --text "same one or two sentences"
```

Site-specific follow-ups — ours, shown as worked examples; adapt or delete:

```text
# external peer bridge (see "Peers" below):
HERMES_HOME="$HOME/.hermes" hermes -p NAME peer add <peer> --url <peer-url> --key "$(head -1 <keyfile>)"
# Honcho memory provider (see "Honcho" below):
HERMES_HOME="$HOME/.hermes" hermes honcho sync
HERMES_HOME="$HOME/.hermes" hermes -p NAME config set memory.provider honcho
```

**Peers — check after every create (site-specific; adapt or delete).** Peer *validation* reads the shared root config; *delivery* reads the *sending profile's* config. A new bot will dispatch `message_agent` to a peer as valid and then die silently in delivery unless the peer is registered on that profile. Register every peer the bot must reach, immediately after create. If your fleet has no external peers, delete this step. Do not print the key.

**Honcho — only if you use the Honcho memory provider.** We never `--clone`, so the profile does **not** get a `hermes_<name>` host block. `hermes honcho sync` backfills (idempotent). Then set `memory.provider honcho` on the new profile — create leaves it empty. Verify: `hermes honcho peers` shows the profile with your user peer and AI peer `NAME`. If `hermes honcho --target-profile NAME status` says not connected, copy `apiKey` / `baseUrl` / `requestTimeout` from the `hermes` host block into `hermes_<name>` in `~/.hermes/honcho.json`. Do not print them. Docs: https://hermes-agent.nousresearch.com/docs/user-guide/features/memory-providers#new-profile-fresh-honcho-peer

From this profile, omit `HERMES_HOME` and `describe default` / `profile create` can hit the wrong home. Always prefix sibling-profile CLI with `HERMES_HOME=$HOME/.hermes`.

Never `--clone` / `--clone-from` / desktop **clone** for a specialist (copies SOUL, skills, memory, leftover local `base_url`). Desktop **create empty** is acceptable. Then `hermes profile describe` — `profile.yaml` is the roster one-liner other bots read; it is not a group-chat log. **Title** (`ui_meta.hermes-bots.title`) is separate — `describe` does not set it.

`SOUL.md` is a protected agent-instruction file. `write_file` will prompt your human. If approval times out, stop and ask — do **not** sneak it in via the shell.

Verify with `hermes -p NAME config get model` — must be **only** `default` + `provider`. Create copies your default profile's custom model block; leaving it makes the "independent" bot talk to the local inference server.

Never hand-edit `config.yaml`. Never copy credentials. Never print `auth.api_key` / `secret_key`.

Then: write signed `SOUL.md`, thin `memories/USER.md` — if the skill is shared, include the **lockstep sentence**: one line stating that the canonical skill lives under `~/.hermes/skills/<category>/<name>/`, the profile files are symlinks, and process changes patch the canonical tree (so later sessions do not re-flag the drift). Then empty-or-tiny `memories/MEMORY.md` (the child fills this). Install the runbook skill. If default Hermes also needs it, link with:

```text
python3 "$HOME/.hermes/skills/autonomous-ai-agents/botmaker/scripts/link_skill_tree.py" \
  "$HOME/.hermes/skills/<category>/<name>" \
  "$HOME/.hermes/profiles/NAME/skills/<category>/<name>"
```

`--no-skills` still seeds essential `hermes-agent` as a real copy. Leave that copy alone; `hermes update` owns it. Extra skills (a notes skill, the child's runbook) are file-level links. Not every specialist needs a custom skill — `@hostadmin` is `hermes-agent` + earned MEMORY. Do not invent a runbook so the folder looks complete.

Stop. Do not fill the child's `MEMORY.md`. Do not add a vault roster row yet.

### D. Certification (child's Bot Chat)

Kickoff: first look at the real box/job → one failure path → client ping if there is a client. The child patches its own skill + `MEMORY.md`. You may `message_agent` one job ("first look") and then get out.

Certified when it has done the job against the live system, rewritten the stale bits of its runbook (if it has one), and pinged the client (or you agreed there isn't one). A first look that finds nothing broken is still a first look — do not invent a failure to practice. Your human saying "document it in the vault" is a ship signal. Uncertified profiles do not get a `Bots/<name>.md` or a roster row unless your human overrides.

### E. Vault

Only after D. See `references/vault.md`.

## Pitfalls

- **Directory symlink** of a skill folder → invisible to rglob on Python 3.11. File-level only.
- **Two copies of a shared skill.** Dual trees drift. Canonical + file links.
- **Create leftover local `base_url`.** Always unset `model.base_url` / `model.api_key` after pin.
- **`config get ui_meta` empty.** Title lives in `profile.yaml` (`ui_meta.hermes-bots.title`). `describe` does not set it.
- **`config get toolsets` is `hermes-cli`.** The spec list is `platform_toolsets.cli`. Don't "fix" toolsets off the wrapper key.
- **Missing peers.** New profiles do not inherit peers. Register every peer the bot must reach immediately after create (`hermes -p NAME peer add …`). Dispatch-success then silent delivery death is this bug.
- **Missing Honcho peer.** (Honcho users.) `--no-skills` create does not add `hermes_<name>` or set `memory.provider`. `hermes honcho sync` then `config set memory.provider honcho`. One of our bots shipped without either and ran memoryless for days before anyone noticed.
- **Alias clobber.** Default wrapper `~/.local/bin/<name>` follows symlinks and can destroy a real CLI. Preflight `zsh -lc 'which -a NAME'` **and** `test -e/-L ~/.local/bin/NAME`. Bare `which` is untrusted (launchd PATH). Hit → `--no-alias`. Landmines: any name matching an installed CLI (it has already happened here), `claude`.
- **Writing the child's MEMORY.md.** Fakes the training loop. Don't.
- **Default Hermes SOUL.md.** Do not touch it.
- **botmaker-2.** Recursion is a bug.
- **Unpinned model.** Same class of bug as cron drift. Pin it.
- **Blind service restart.** Not your job; if you are tempted, you are the wrong bot.
- **Model-identity costume.** Running on Grok does not make the bot Grok; running on Claude does not make it Claude. Lead the SOUL with the job name. `@gpuops` shipped as "You are Grok" in a hat and had to be rewritten down to its job; do not regress a shipped bot into costume.
- **Absorbing the adjacent job.** `@gpuops` does not mint bots. You do not operate the inference server or ComfyUI. There is no doctor class — a specialist's runbook includes diagnosis of *its* job. You diagnose the fleet and this process.
- **UI clone / `--clone`.** Costumes and leftover local `base_url`. Create empty or `profile create --no-skills`.
- **`profile.yaml` essays.** Roster one-liner via `hermes profile describe`. No chat logs. `@persona` is a persona — not a `Bots/` note. From a named profile, `hermes profile describe default` writes *this* `$HERMES_HOME` — set `HERMES_HOME=~/.hermes` to hit default. Same prefix for `profile create` / `-p NAME` against siblings.
- **Unsigned `SOUL.md`.** `SOUL.md` is a protected agent-instruction file; writes normally prompt your human — that prompt is the human gate materialized, and the default (`security.protected_instruction_files: true`) is the right default. We disabled it on this one profile only because a headless worker burned its whole approval window on a prompt nobody could see and shipped a stub. Understand what the flag removes: the tooling gate. The rule survives the flag — the gate moved to the signature. Write a child SOUL only from a human-signed draft or signed spec. If a write still gets blocked, stop; do not `cat` around it.
- **Injected skills must be discoverable.** Any automation that launches a worker with `--skills <name>` (our review lane does) crashes at init on a `--no-skills` profile (`Unknown skill(s): <name>`) until the canonical skill is file-level linked into the profile. That is availability, not a profile pin. Do not always-load it.
- **Invented runbook.** Experiential specialists (`@hostadmin`) do not get a fake skill tree. `hermes-agent` + MEMORY is enough until the box teaches a procedure.
- **Invented failure drill.** First look with nothing to repair still counts. Client ping after a real fix.
- **`skill_manage` unlinks.** Patching a profile file-symlink materializes a second copy. Patch the canonical path with the file `patch` tool, then `ls -l` — if the profile `SKILL.md` is not an `l`, relink with `scripts/link_skill_tree.py`. `skill_manage` `file_path` on references also fails (path escapes the profile skill dir).
- **`skill_view` `file_path` on file-level skill symlinks.** Same escape as `skill_manage` `file_path`. `skill_view(name)` on `SKILL.md` works; `skill_view(name, file_path="references/…")` follows the file symlink into `~/.hermes/skills/…` and `validate_within_dir` rejects it. Workaround: `read_file` the canonical `~/.hermes/skills/<cat>/<name>/…` path. Do not tell bots to `skill_view` linked references until Hermes allows it.

## Verification

A new bot is done when:

1. `hermes profile list` shows it on the pinned model, not the stock default model.
2. `hermes -p NAME config get model` is only provider + default.
3. `.no-bundled-skills` exists in the profile root.
4. (If your fleet has peers) `hermes -p NAME peer list` shows every peer it must reach — or DMs will ack then vanish.
5. (Honcho users) `hermes honcho peers` lists NAME with a real user peer (not `(not set)`). `memory.provider` is `honcho`.
6. `SOUL.md` matches the signed draft (one screen).
7. Shared skills: `ls -l` on profile `SKILL.md` shows `l` (symlink), canonical is a regular file. `rglob` from the profile skills dir finds `SKILL.md`.
8. After certification only: vault note + roster row + Home line + `making-bots.md` changelog.
