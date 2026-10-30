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
| T4a Branding slice: CLI help, banner and user-facing strings say Atreus while the package/npm identity stays `gentle-pi` | S4, S5, L15 | parent inline | blocked by L16 (slice budget) |
| T4b Package/npm identity rename (`package.json.name`, bin entries, provider install target compat) | S4, S5, L15 | pending the publish decision | blocked: needs owner call on publishing `atreus` |
| T5 Open-source hygiene: front README (states the fork provenance), `SECURITY.md` with "inbound messages are untrusted", `CONTRIBUTING.md`, issue templates kept, fork-safe CI | S1, F6, F7, L11 | parent inline | done, see Log L12-L13 |
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
- **L10** (finding, read-only pass before T4): the product identity `gentle-pi` is
  wired in four places that must move together, or the fork breaks its own
  install/update path:
  `lib/gentle-shell-launcher.ts:594` (`npm:gentle-pi` source detection),
  `lib/gentle-shell-launcher.ts:659` (`{ name: "gentle-pi", source: "npm:gentle-pi" }`
  package declaration), `lib/gentle-shell-launcher.ts:676`
  (`LAUNCHER_EXCLUDED_PI_PACKAGES`), `lib/gentle-shell-bundled.ts:113` (realpath
  check for `node_modules/gentle-pi`), plus `bin/gentle-shell.mjs:1250,1260,2164`
  (update source and declaration messaging).
  `extensions/gentle-shell.ts:1708,1727` also send `User-Agent: gentle-pi` to model
  providers — a network-visible identity string, so the fork must decide whether
  Atreus identifies itself honestly in traffic or keeps the upstream UA for
  compatibility. Rename slice must keep a compatibility path for the old
  `npm:gentle-pi` specifier so existing installs still resolve.
- **L11** (baseline evidence): `pnpm test` on the untouched fork at `3ee1d90` gave
  `PASS provider-contract`, `PASS runtime-harness`, `FAIL unit-tests`. The unit
  failure is an environment leak, not a fork defect: this session exports
  `GENTLE_SHELL_CONFIG_HOME`, `GENTLE_PI_AGENT_HOME` and `PI_CODING_AGENT_DIR` to the
  ambient gentle-shell home, and `tests/agent-home.test.ts:16` isolates only
  `HOME`/`USERPROFILE`/`GENTLE_PI_CONFIG_HOME`, so `gentlePiConfigHome()` returned
  `/home/sistmas/.gentle-shell/gentle-ai` where the test expected the temp config
  home. With those variables unset, `tests/agent-home.test.ts` is green (6/6).
  Consequence: any T4 verification must run in a clean env, and the caveat is
  documented in `CONTRIBUTING.md`.
- **L12** (2026-10-30, T5): README now opens with the Atreus identity, the
  nominative provenance line required by S5, and a pointer that the inherited
  gentle-shell surface below is upstream material for task T4. Added `SECURITY.md`
  (report route, owned vs inherited supply chain, untrusted-inbound posture for the
  gateway phase) and `CONTRIBUTING.md` (dev loop including the L11 env caveat, ODD
  task-document convention, ~400-line review slices, identity boundaries).
- **L13** (repo settings observed, not assumed): `leav-dev/atreus` has Actions
  enabled (`allowed_actions: all`) with **zero workflow runs so far** — CI evidence
  waits for the first push. Issues were **disabled by default on the fork** and were
  enabled. Private vulnerability reporting could not be enabled over the API
  (`PUT /vulnerability_alerts` → 404), so `SECURITY.md` carries the report route.
  `tests/agent-home.test.ts` passes 6/6 in a clean env; `tests/gentle-ai.test.ts`
  passes 154/154 after the README change, which keeps the
  `docs/readme-reference.md` link asserted at `tests/gentle-ai.test.ts:937`.
  The full unit stage still fails in a clean env and is under diagnosis.
- **L14** (baseline evidence, 2026-10-30): the clean-env unit stage is **green**.
  `node --experimental-strip-types --test tests/*.test.ts` with
  `GENTLE_SHELL_CONFIG_HOME`, `GENTLE_PI_AGENT_HOME`, `PI_CODING_AGENT_DIR`,
  `GENTLE_SHELL_CHILD_PACKAGE_INJECTION`, `GENTLE_SHELL_USER_PI_HOME`,
  `PI_SESSION_FILE` and `PI_SESSION_ID` unset: **7989 tests, 7910 pass, 0 fail, 79
  skipped**, exit 0, in 181s. The `pnpm test` FAIL in the earlier clean-env run was
  my own concurrent README edit landing mid-suite, not a fork defect. Conclusion:
  the fork inherits a green baseline on this machine when the harness env is
  stripped, so T4 can be judged against real evidence.
- **L15** (finding, blocks the T4 package-name half): the `gentle-pi` string is not
  only our product name, it is also the **provider's install target**. The fork's
  own bundled-install detection resolves `node_modules/gentle-pi`
  (`lib/gentle-shell-bundled.ts:113`), and the launcher both removes
  `npm:gentle-pi` after the provider's setup
  (`lib/gentle-shell-launcher.ts:659`) and tells the pinned provider to skip it
  (`lib/gentle-shell-launcher.ts:676`). The pinned `gentle-ai` v4.3.1 binary still
  installs the upstream `gentle-pi` Pi package into homes, so renaming
  `package.json.name` to `atreus` without publishing `atreus` splits identity
  across two names: a checkout that calls itself Atreus and a home where the
  provider drops upstream `gentle-pi`. The package-name rename therefore belongs
  with the publish decision, not with the branding slice.
- **L16** (sequencing): slice 1 (identity contract + open-source hygiene) is at 399
  authored lines, so it is already one reviewable unit and no room is left under the
  ~400-line budget. T4a and everything after it wait for slice 1 to land (push/PR,
  owner decision), rather than inflating one branch past reviewable size.
- **L17** (slice boundary, recorded at the 400-line crossing): **slice 1 = commits
  `a1a688388..42700b6bc`** on `feat/atreus-bootstrap` (9 commits, 404 authored lines,
  docs and repo policy only, no behavior change). Slice 2 onward starts from `main`
  only after slice 1 has a delivery decision, and slice 2 is T4a (CLI/banner/help
  strings, which also has to move the tests that assert them).
- **L18** (native review of slice 1, approved and burned): assess read
  `nativeRisk high` (`hot_path` security signal on `SECURITY.md`) with
  `writerProfile large` from runtime, so this candidate earned a review. Ordinary
  review ran the two lenses the change touched — `review-risk` and
  `review-readability` — on tier high, frozen budget 200 correction lines over 409
  changed lines, and closed **approved** on the last admitted event. Receipt: lineage
  `review-ad6a163b0ca56f83`, target `sha256:364fe350…`, consumed revision
  `sha256:0fca8144…`, acknowledgement `gentle-ai.review-acknowledged/v1`, authority
  **burned**, delivery left to ordinary repository policy.
  Four **advisory, non-blocking** findings stay as later work and must not reopen
  this candidate: `NOTICE.md:8-10`, `SECURITY.md:9-11`, `CONTRIBUTING.md:23-24`
  (readability WARNINGs) and `README.md:25` (SUGGESTION). They are wording/prose
  items for the T4a documentation pass.
