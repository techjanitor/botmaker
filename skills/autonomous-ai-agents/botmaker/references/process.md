# Process

Interview → SOUL draft → human sign-off → scaffold → child's Bot Chat (first look, failure path, client ping) → child rewrites runbook/MEMORY → vault.

You draft. The child earns. If you fill `MEMORY.md`, you faked the training loop.

## Independence

If the bot operates a service — CLI, HTTP API, GUI control socket, ComfyUI, whatever — **do not run the bot on that service's inference path.**

`@gpuops` is the proof: our default Hermes pointed its model at the local inference server, so default Hermes died whenever that server died. The operator that *manages* the server is pinned to a hosted provider instead. A ComfyUI specialist would be the same class of thing: talks *to* the API; must still think when the GPU box is busy or down.

Same machine is fine; same inference path is not.

Pin `model.provider` / `model.default` with `hermes -p NAME config set`. Never hand-edit `config.yaml`. Do not copy `auth.json` between profiles — shared pools already rotate. Never print `auth.api_key` / `secret_key`.

**Create leftover:** `hermes profile create` copies your default profile's custom model block (e.g. `base_url: http://127.0.0.1:8000/v1` plus an api_key env ref). After pinning the hosted provider you **must** unset those two keys. Verify:

```text
hermes -p NAME config get model
```

Must print only `default` and `provider`. If `base_url` still points at the local server, the "independent" bot is not.

**External peers after create (site-specific; adapt or delete).** Validation uses root config; delivery uses the sending profile. Register every peer the bot must reach on every new bot:

```text
hermes -p NAME peer add <peer> --url <peer-url> --key "$(head -1 <keyfile>)"
```

Do not print the key.

**Honcho peer after create (only if you use the Honcho memory provider).** Never `--clone`, so no fresh AI peer. Then:

```text
hermes honcho sync
hermes -p NAME config set memory.provider honcho
```

Verify `hermes honcho peers`. If status is not connected, inherit `apiKey`/`baseUrl`/`requestTimeout` from host `hermes` into `hermes_<name>`. Do not print them.

## --no-skills

```text
hermes profile create NAME --no-skills --description "one or two sentences"
```

**Alias preflight before that command.** `zsh -lc 'which -a NAME'` and `test -e "$HOME/.local/bin/NAME" || test -L "$HOME/.local/bin/NAME"`. Bare `which` is untrusted (launchd PATH). If either hits: `--no-alias`, or a custom alias only after the same two checks. `~/.local/bin/NAME` as a **symlink** is the binary-destroying case (`Path.write_text` follows; it has destroyed a real CLI binary here). Landmine: `claude`. Hermes only reserves `hermes`, `default`, `test`, `tmp`.

Never `--clone` / `--clone-from`. Desktop clone is the same bug. Desktop create-empty is fine.

Writes `.no-bundled-skills`. `hermes update` will not dump the bundle on this profile. Essential `hermes-agent` still seeds as a **real copy** — leave it; sync owns it.

A review-lane bot still needs its review skill **discoverable** on the profile even when it is not pinned. Any automation that injects `--skills <review-skill>` dies at init on a `--no-skills` tree that cannot resolve the name (`Unknown skill(s): <review-skill>`). Link the canonical skill file-level. Do not always-load it.

Extra skills are context tax and off-mission bait. Three is already a lot (`botmaker` itself: its own runbook + `hermes-agent`, plus a notes skill if you keep a vault).

## Shared skills (file-level links)

Canonical tree: `~/.hermes/skills/<category>/<name>/` (default Hermes loads this).

Profile path: `~/.hermes/profiles/<bot>/skills/<category>/<name>/` — **real directories**, every *file* a relative symlink.

Never symlink the skill directory itself. Hermes venv is Python 3.11; `Path.rglob("SKILL.md")` does not descend directory symlinks → empty index → the bot cannot see the skill. File-level `SKILL.md` inside a real dir **is** found.

`skill_manage` delete refuses if the skill **dir** is a symlink (will not `rmtree` through a redirect). Sandbox skill mounts skip symlinks — acceptable for a runbook the bot reads; do not put secrets in the skill either way.

Linker (this skill):

```text
python3 "$HOME/.hermes/skills/autonomous-ai-agents/botmaker/scripts/link_skill_tree.py" \
  "$HOME/.hermes/skills/<category>/<name>" \
  "$HOME/.hermes/profiles/<bot>/skills/<category>/<name>"
```

Writes through the profile path hit the same inode. If a rewrite unlinks first, `ls -l` shows a regular file instead of `l` — that is a second copy starting again. Relink; do not keep both. `skill_manage` patch on this profile's linked `SKILL.md` does that unlink. Patch canonical with the file `patch` tool instead. `skill_manage` `file_path` and `skill_view(..., file_path=…)` on `references/` both fail: `Path.resolve()` follows the file symlink into `~/.hermes/skills/…`, then `relative_to` the profile skill dir rejects it. Workaround: `read_file` the canonical path. Do not tell bots to `skill_view` linked references until Hermes allows it.

`$HERMES_HOME` inside a named profile is `~/.hermes/profiles/<name>`, not `~/.hermes`. Canonical skills stay under `$HOME/.hermes/skills/`. Sibling-profile CLI (`profile create`, `-p NAME config`, `profile describe`) from this bot: prefix `HERMES_HOME=$HOME/.hermes`.

`SOUL.md` writes on a child profile normally prompt your human — that prompt is the human gate materialized, and the default (`security.protected_instruction_files: true`) is the right default. If your fleet has disabled it for headless workers, authority comes from the signed spec, not the tooling: no signature, no write. If a write still gets blocked (flag reverted, running as another profile), stop. Do not use the terminal to bypass.

`hermes profile describe` does not set the roster **title**. Write `ui_meta.hermes-bots.title` (e.g. HostAdmin) after describe.

## Generate-the-experience

Do not hand the child a finished runbook and a stuffed MEMORY.

Kickoff in the child's Bot Chat:

1. First look at the real box or job (status script, live files, not vibes).
2. One failure path, practiced (the thing that will hurt at 2am).
3. Client ping if there is a client (`@hermes` was `@gpuops`'s check that bot-to-bot actually works). First DMs can fail with `open terminal failed: not a terminal` — tmux / no-TTY, then retry.

The child patches its own skill + MEMORY. You may send one `message_agent` ("first look") and then get out.

Certified when it has done the job against the live system and rewritten the stale bits. A clean first look is enough if your human ships it ("document it in the vault"). Uncertified profiles do not get a roster row unless your human overrides. Do not invent a 2am failure to practice.

## Bot-to-bot

Useful as a **health primitive** (can this bot reach another bot?) and as a client-path check. Not as a substitute for the child doing the work.

Do not impersonate children. Do not DM them a finished MEMORY.

## Blast radius

v1 Botmaker: specialists + living guide + this skill. You are the bot coordinator; `@hermes` is not. Touch existing bots' SOULs (ours: `@gpuops` / `@persona` — substitute your fleet) only when your human explicitly sends you there. Never default Hermes SOUL. There is no doctor class — each specialist diagnoses its own job; you diagnose the fleet.

Do not spawn `botmaker-2`.
