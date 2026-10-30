# Atreus bootstrap — OpenClaw-style harness forked from gentle-shell

Atreus is a public fork of `Gentleman-Programming/gentle-shell` whose target is a
self-hosted agent that can actually do many things, running on a laptop or a VPS,
while keeping gentle-shell's engineering discipline (ODD, scoped subagents,
permission guards, review). The fork is real and the migration is progressive:
identity and open-source posture first, then the gateway runtime that turns the
harness from an interactive TUI into an always-on service.

## Specs

- **S1** (verbatim, owner request 2026-10-30): "vamos a hacer un nuevo proyecto se
  quiere un harness estilo openclow, pero con la mentalidad de mi, y basandonos en
  los repos de gentle-shell y openclow para avanzar en el proyecto, este prpyecto
  sera de codigo abierto desde su inicio"
- **S2** (verbatim, MVP thesis): "la idea es poder levantar un agente que pueda
  hacer muchas cosas levantado ya sea en local o en una vps"
- **S3** (verbatim, repo): "git@github.com:leav-dev/atreus.git este seria el repo"
- **S4** (owner decisions from the bootstrap questionnaire): fork path = "Fork
  real, migrar progresivo"; turn executor = "Pi SDK / one-shot por turno";
  identity/home = "atreus bajo tu cuenta"; topology = "Fork GitHub real"; first
  slices = "Identidad Atreus" + "Higiene open source"; first channel = "Loopback
  (HTTP+CLI)".
- **S5** (upstream trademark constraint, `TRADEMARKS.md:33` verbatim): "Modified
  distributions and forks must use distinct primary names and branding. They may
  truthfully state that they are based on or compatible with gentle-shell or
  gentle-pi, provided that the statement does not imply that the distribution is
  official, sponsored, endorsed, certified, or partnered."

## Findings (evidence from the read-only pass)

| # | Finding | Evidence |
| --- | --- | --- |
| F1 | The fork base is a Pi package, not a standalone runtime: product name is `gentle-pi`, CLI is `gentle-shell`, and Pi loads its extensions/themes/prompts/skills. | `package.json` (`name`, `bin.gentle-shell`, `pi` block) |
| F2 | Review/RDD is delegated to an external Go binary that this repo does not own, installed by a script. Any Atreus runtime inherits that dependency boundary. | `scripts/install-gentle-ai.mjs:8` (`installGentleAi`) |
| F3 | Brand surface is wide but unevenly owned: `gentle-shell` 330 files / 2505 lines, `gentle-pi` 380 files / 2287 lines, `gentle-ai` 528 files / 4796 lines. The `gentle-ai` surface is the external provider contract, so it is out of scope for a rename. | `git grep -c -i` census on `main` |
| F4 | Headless turns are supported by Pi's own SDK: `createAgentSession()`, `SessionManager` (persistent or in-memory), `tools`/`customTools`/`resourceLoader` boundaries, `agent_settled` for "Pi will not continue", and `cwd` for per-workspace discovery. This is what an always-on gateway needs. | pi `docs/sdk.md` |
| F5 | OpenClaw's shape to emulate (MIT, reference only): one long-lived local Gateway owns every channel connection; CLI/TUI/UI/nodes are control-plane clients over WebSocket; inbound messages are treated as untrusted input; models and agent harnesses are swappable plugins. | `openclaw` README + `docs.openclaw.ai/concepts/architecture` |
| F6 | The npm publish workflow cannot fire from this fork: it is gated to the upstream repository. The fork starts without a publishing hazard on the upstream package name. | `.github/workflows/publish.yml:16` (`if: github.repository == 'Gentleman-Programming/gentle-shell'`) |
| F7 | Inherited CI runs on `push: main` and every `pull_request`; Dependabot and stale-issue automation came with the fork. | `.github/workflows/ci.yml:3-7`, `.github/dependabot.yml` |
| F8 | Upstream keeps its own ODD tracking in `odd/{tasks,prs,notes,diagnostics}`; Atreus tracking continues in the same layout so the history stays readable. | `ls odd/` |

## Tasks

| ID | Spec | Route | Commit |
| --- | --- | --- | --- |
| T1 Seed the real GitHub fork of gentle-shell as public `leav-dev/atreus` | S3, S4 | parent (GitHub API) | none — fork created server-side; `origin/main` == upstream `3ee1d90`, 6510 commits, 87 tags |
| T2 Write this bootstrap specification | S1-S5 | parent inline | `feat/atreus-bootstrap` commit for this file |
| T3 Identity contract: `docs/identity.md` + `NOTICE.md` naming what Atreus owns (primary name, CLI, package, logo) and what stays external (`gentle-ai` provider, Pi), with the nominative "based on gentle-shell" wording | S4, S5, F1-F3 | parent inline | see Log L7 |
| T4 Identity rename slice 1: rename only Atreus-owned surface (`package.json name`, `bin/atreus`, launcher, banner, docs front page) with tests alongside; keep `gentle-ai` provider paths untouched | S4, S5, F2, F3 | `gentle-ai-worker` (background), surfaces `package.json`, `bin/**`, `lib/gentle-shell-launcher.ts`, `runtime/**` | pending |
| T5 Open-source hygiene: front README (states the fork provenance), `SECURITY.md` with "inbound messages are untrusted", `CONTRIBUTING.md`, issue templates kept, fork-safe CI | S1, F6, F7 | parent inline first, then worker if the diff grows | pending |
| T6 Architecture decision record for the gateway: loopback HTTP + CLI client, one Pi SDK session per turn, session persistence keyed by conversation, ODD authorization layer as the guard | S2, S4, F4, F5 | parent inline (decision doc), worker for code later | pending |
| T7 Walking skeleton: daemon that takes a loopback POST, runs one headless Pi SDK turn under the Atreus identity, returns the reply; deterministic test | S2, F4 | `gentle-ai-worker` after T6 | pending |

## Log

- **L1** (owner, verbatim): "vamos a hacer un nuevo proyecto se quiere un harness
  estilo openclow, pero con la mentalidad de mi, y basandonos en los repos de
  gentle-shell y openclow para avanzar en el proyecto, este prpyecto sera de codigo
  abierto desde su inicio"
- **L2** (owner, MVP thesis, verbatim): "la idea es poder levantar un agente que
  pueda hacer muchas cosas levantado ya sea en local o en una vps"
- **L3** (owner, repository, verbatim): "git@github.com:leav-dev/atreus.git este
  seria el repo"
- **L4** (decision, 2026-10-30): `leav-dev/atreus` was empty and public. It could
  not be attached to the fork network in place, so it was renamed to
  `leav-dev/atreus-empty-placeholder` (name freed) and `gh repo fork
  Gentleman-Programming/gentle-shell --fork-name atreus` created the real fork.
  Deleting the placeholder needs the `delete_repo` OAuth scope, which the current
  token does not have — owner cleanup, pending.
- **L5** (constraint): upstream `TRADEMARKS.md` (F/S5) forces Atreus to ship a
  distinct primary name and branding; it does not block "based on gentle-shell"
  phrasing. MIT license and upstream copyright notices stay in the tree.
- **L6** (finding): the `gentle-ai` namespace is a provider contract, not a brand
  we can rename (F2, F3). Identity migration must scope to Atreus-owned surface.
- **L7** (2026-10-30, T3 done): `docs/identity.md` + `NOTICE.md` committed on
  `feat/atreus-bootstrap`. Two extra contract surfaces found while writing it:
  `lib/gentle-shell-launcher.ts:593-595` detects `npm:gentle-pi` in Pi settings, so a
  package rename needs a compatibility specifier; and the assistant persona/system
  prompt comes from the external provider state (`~/.gentle-shell/gentle-ai/persona.json`,
  mode `gentleman`), which means a rename alone does not change how Atreus speaks —
  the "mentalidad" needs an Atreus-owned prompt layer or provider support.
- **L8** (owner decision, verbatim): "vamos a usar por defecta una version
  especifica, no pretendemos ir a la par de gs, podemos usar ahora la version mas
  reciente gs". Disposition: Atreus pins one `gentle-ai` provider version and does
  not ride upstream releases. The pin is already `INSTALLER_VERSION = "4.3.1"`
  (`scripts/gentle-ai-installer.mjs:40`) with per-platform release digests frozen
  in `GENTLE_AI_RELEASE_ASSETS` (`scripts/gentle-ai-installer.mjs:118`), and
  upstream `v4.3.1` is the latest release today, so no version change was needed.
  `scripts/verify-package-files.mjs:323` fails if `lib/gentle-ai-binary.ts` drifts
  from the installer pin, so bumps stay explicit commits.
- **L9** (owner decision): persona stays `gentleman` for this phase — Atreus keeps
  the inherited voice and ships no Atreus-owned prompt layer yet. Identity work is
  product surface only (name, CLI, docs, provenance).
