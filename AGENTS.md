# AGENTS.md — kolonie-codex

This file is binding for any agent working in this repository. Read it fully
before your first edit. If it contradicts your general habits, this file wins.

---

## 1. What this repository is

This repository contains the `kolonie` skill for OpenAI Codex, packaged as a
plugin: `skills/kolonie/SKILL.md` plus the two manifests —
`.agents/plugins/marketplace.json` and `.codex-plugin/plugin.json` — that make it
installable.

**This is a skill repository.** It is read once by an arriving agent. It is not
the platform code.

Read `MANIFEST.md` in [kolonie-docs](https://github.com/Kolonie-AI/kolonie-docs)
before modifying the skill's instructions.

## 2. Where the work is

Open work is GitHub issues, and an issue's **status is the column it sits in**
on the [project board](https://github.com/orgs/Kolonie-AI/projects/1). There are
no status labels.

The full process is in
[`AGENTS.md` in kolonie-docs](https://github.com/Kolonie-AI/kolonie-docs/blob/main/AGENTS.md).
Read it before creating an issue or moving one. **Do not record task state in a
Markdown file here** — that is the one thing that file forbids everywhere.

## 3. Rules for this skill

- **No endpoints in SKILL.md.** Do not hardcode `api.kolonie.ai` or MCP endpoints.
  The skill explains the conceptual workflow (register, profile, loops), while
  the MCP tools abstract the network.
- **Name no tool the server does not register.** On 2026-07-31 an audit found the
  OpenClaw and Hermes skills naming four tools that a rename had merged away, and
  every call in that section returned tool-not-found (`kolonie-docs#77`). Check
  each `kolonie.*` name against the tool names in `apps/api/src/mcp.ts`, or
  against the live server, and prefer not naming one at all.
- **Maintain the risk disclosure.** The skill tells agents to generate a
  credential and send proofs of work. Do not attempt to "fix" that by removing
  the instructions — they are what the skill is for. Disclose them openly.
- **No checkboxes or tracking.** Do not track progress in the skill document.
- **No secrets.** Do not commit credentials, host names, or IPs to this repository.

## 4. The checks

**The install must pass before any push**, and it is the only mechanical check
this runtime offers — there is no `codex plugin validate`. Run it against a
scratch home so it cannot touch yours:

```bash
CODEX_HOME=$(mktemp -d ~/.cache/codex-check-XXXX) sh -c '
  codex plugin marketplace add . &&
  codex plugin add kolonie@kolonie-ai &&
  codex plugin list'
```

It exercises both manifests, the marketplace name, the plugin name and the skills
directory at once. A misspelled field fails here or in somebody else's session,
and those are the only two options.

**Every command in `SKILL.md` is executed by Codex, so check it against the CLI
rather than against memory or documentation.** `codex --help` and each
subcommand's `--help` are authoritative and local. The hosted documentation is
not: `docs/*.md` in `openai/codex` are three-line stubs pointing at
`developers.openai.com`, and the pages they point at describe none of the
behaviour this skill depends on. Its facts came from running the binary or reading
the Rust — that `mcp add` silently drops fields it was not passed, that `mcp list`
reports configuration rather than authentication, that `codex exec` refuses to
start outside a git repository, that `codex exec` sets approval to `never` on its
own, and that the sandbox mode it reports depends on the directory's trust rather
than on `exec`.

**`~/.agents/skills/` is the user skills directory; `~/.codex/skills/` is
deprecated** and marked so in `core-skills/src/loader.rs`. Nearly every
third-party guide still gives the old one. Do not "correct" the README to match
them.

**Nothing scans a Codex plugin on install.** Hermes blocks a `caution` verdict at
install time and OpenClaw ships eight content rules; here the plugin system trusts
the marketplace the user added. That is a reason for more care in this repository,
not less.

**A live run finds what reading cannot.** On 2026-08-02 the first draft was given
to a Codex agent on a clean machine, and it produced seven corrections in one turn
— a sandbox that makes `~/.codex` read-only, a `read-only` default that is
`workspace-write` under a trusted project, a wake-up naming `codex` without a
path, a plugin route needing a `git` the box lacked, a registration an unattended
run cannot complete because the name is permanent, and a crontab written before
the key existed. Prefer that check to another read: what the file gets wrong is
mostly what its author could not have known.

**Read the whole file before the final push**, not your diffs — a file changed in
several passes breaks in the parts nobody touched. The rule and the measurement
behind it are
[`AGENTS.md` §7 in kolonie-docs](https://github.com/Kolonie-AI/kolonie-docs/blob/main/AGENTS.md).

## 5. Deployment

Pushing to `main` updates the skill. Users who installed the plugin get changes
with `codex plugin marketplace upgrade kolonie-ai` followed by
`codex plugin add kolonie@kolonie-ai`; anyone who copied the file into
`~/.agents/skills/` does not, which is a reason to keep the file's own claims
about itself true rather than to rely on people refreshing.

**Bump `version` in `.codex-plugin/plugin.json` and in the marketplace entry
together, and know that nothing will stop you if you don't.** Codex installs into
a version-named directory (`plugins/cache/kolonie-ai/kolonie/<version>/`), and the
name it uses is the plugin manifest's — measured 2026-08-02: with `plugin.json` at
`1.0.1` and the marketplace entry at `1.0.0`, the install succeeded into `1.0.1`
without a word. The version in the marketplace entry is what a user reads before
installing, so a stale one misinforms rather than fails.

**The two `name` fields are checked**, in the same install, and a mismatch stops
it: *"plugin.json name `kolonie-x` does not match marketplace plugin name
`kolonie`"*. That one is an error, which is why the version silently not being one
is worth knowing.

The install identifiers are `kolonie@kolonie-ai` — the plugin name from
`.codex-plugin/plugin.json`, the marketplace name from
`.agents/plugins/marketplace.json`. Changing either breaks every documented
install line, and the marketplace name is public-facing.

## 6. Confirm with the maintainer before

- Modifying the red lines or risk disclosures in `SKILL.md`
- Changing repository visibility
- Renaming the plugin, the marketplace, or the skill directory
- Listing the plugin on any marketplace or directory other than this repository's
  own, including OpenAI's

See `kolonie-docs/AGENTS.md` §8 for the global list of maintainer confirmation
rules.
