# Atreus identity contract

Status: slice 1 (naming boundary only). Task: `odd/tasks/atreus-bootstrap.md`, T3.
Specs: S4 (owner decisions), S5 (upstream trademark constraint). Findings: F1-F3, F6.

Atreus is a public fork of `Gentleman-Programming/gentle-shell` at `3ee1d90`
(2026-10-30). Upstream code stays MIT with its copyright notices intact; see
[NOTICE.md](../NOTICE.md). This file defines what the fork is allowed to rename
and what it must not, so the identity migration does not silently break external
contracts.

## Primary identity

| Surface | Value | Notes |
| --- | --- | --- |
| Project name | **Atreus** | primary name in docs, README first line, banners |
| Repository | `leav-dev/atreus` | public, real GitHub fork of `gentle-shell` |
| CLI (target) | `atreus` | replaces `gentle-shell` for our own invocations |
| Package (target) | `atreus` | replaces npm name `gentle-pi`; **not published** — upstream `publish.yml` is gated to the upstream repository (F6), so the fork cannot ship the upstream package name |
| License | MIT (upstream notice retained) + `NOTICE.md` attribution | no new license grants |

## Namespace boundary

**Owned by Atreus, renameable.** These are the fork's own product surface: the
package `name` and `bin` entries, `bin/gentle-shell.mjs`, `lib/gentle-shell-launcher.ts`,
`runtime/gentle-shell-launcher.mjs`, launcher/banner strings, README and docs
front pages, plus everything new Atreus writes (gateway, channels, policy layer).
Census on `main`: `gentle-shell` in 330 files / 2505 lines, `gentle-pi` in 380
files / 2287 lines.

**External contracts, never renamed by a branding pass.**

1. `gentle-ai` (528 files / 4796 lines) is the Go provider that supplies native
   review and RDD. It is resolved by `lib/gentle-ai-binary.ts` (`resolveGentleAiBinary`,
   release-asset mapping) and invoked by `lib/native-review-cli.ts`; its mirrored
   contract is drift-checked offline against
   `contracts/review-provider-contract-mirror/` by `scripts/check-provider-contract.mjs`.
   Renaming any of this detaches native review and fails the contract check.
2. Pi itself (`@earendil-works/pi-coding-agent`) and its SDK surface.
3. npm package detection: `lib/gentle-shell-launcher.ts:593-595` recognises
   `npm:gentle-pi` in Pi settings. Renaming our package without a migration
   detection for the old specifier breaks installs that still reference it.
4. Runtime state paths (`~/.gentle-shell`, `~/.pi/gentle-ai`, and the persona
   config `~/.gentle-shell/gentle-ai/persona.json`) are created by the provider
   install. Slice 1 keeps them as-is; a fork-owned state path is a separate,
   breaking migration.
5. Upstream `odd/` tracking history, `LICENSE`, copyright lines, and `TRADEMARKS.md`
   in history — retained, not rewritten.

## Rename rules (for T4)

- One pass per surface, each with its tests updated in the same commit.
- Never search-and-replace `gentle-ai`: it is a dependency, not a brand (F2).
- Keep a compatibility alias for the old CLI/bin name until the fork stops
  tracking upstream.
- Every user-facing string that names the product says Atreus; every statement
  about provenance uses the nominative wording below.

## Trademark posture

`TRADEMARKS.md` in upstream requires a distinct primary name and branding for
forks, while permitting truthful nominative reference. Approved wording:

> Atreus is an independent open-source project, based on
> [gentle-shell](https://github.com/Gentleman-Programming/gentle-shell). It is not
> official, sponsored, endorsed, certified, or partnered with gentle-shell or
> gentle-pi, which are marks of their owner.

Do not use upstream logos or brand assets as Atreus primary branding.

## Open identity decisions

Tracked in `odd/tasks/atreus-bootstrap.md` (owner decision, blocking T6):

- Whether Atreus keeps `gentle-ai` as its review/RDD provider, or treats native
  review as optional and gates headless turns with its own policy layer.
- Whether Atreus ships its own persona/system prompt. The injected orchestrator
  prompt and persona come from the external provider, so a package-level rename
  does not change how the assistant speaks: the "Atreus mindset" requires either
  upstream persona support or an Atreus-owned prompt layer.
