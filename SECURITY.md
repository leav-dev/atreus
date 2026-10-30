# Security policy

Atreus is a public fork of gentle-shell (see [NOTICE.md](NOTICE.md)). This page
states what Atreus is responsible for, what it inherits, and how to report a
problem.

## Report a vulnerability

Use GitHub's private vulnerability reporting on `leav-dev/atreus`
(Security → Report a vulnerability), or open a normal issue only if the problem is
already public. Do not open a public issue for an exploitable flaw.

## What this project is responsible for

- The Atreus code in this repository and its own supply chain.
- Dependencies are installed under a frozen supply-chain policy
  (`pnpm-workspace.yaml`): `minimumReleaseAge: 4320` (nothing published in the last
  72 hours), `trustPolicy: no-downgrade`, `blockExoticSubdeps: true`, and an
  `allowBuilds` list that gates lifecycle scripts.
- Release integrity for the native provider: the `gentle-ai` binary is pinned to one
  explicit version with frozen per-platform SHA-256 digests
  (`scripts/gentle-ai-installer.mjs:40,118`), and `scripts/verify-package-files.mjs`
  fails if the consumer drifts from that pin.

## What this project inherits and does not audit

- **Pi** (`@earendil-works/pi-coding-agent`) is the agent runtime. Runtime flaws
  belong upstream.
- The **`gentle-ai` native provider binary** is built and released by its own
  project. Atreus pins and verifies its digest; it does not review its source.
- **Model providers and chat platforms** you configure. Prompts and tool output
  leave your machine under their terms, not ours.

## Security posture for the gateway phase

The target architecture is an always-on agent reachable from message channels.
These are design requirements, not current guarantees:

1. **Inbound messages are untrusted input.** A channel message is not an operator
   instruction and never authorizes a destructive action by itself.
2. **The gateway is the only listener.** It binds to loopback unless the operator
   explicitly exposes it, and remote exposure is a documented, opt-in step.
3. **Tool execution is policy-gated**, with the ODD authorization step (read-only
   until a change is authorized) rather than an approval prompt the agent can
   reinterpret.
4. **Credentials and state stay on your hardware.** No hosted service, no paid
   tier, no account required.

Anything in this list that is not yet enforced is a gap; open an issue if you are
planning work around one.
