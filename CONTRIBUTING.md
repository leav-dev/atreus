# Contributing to Atreus

Atreus is open source from its first commit. It is a fork of gentle-shell with its
own goal: an agent that can do many things, running on your laptop or a VPS, with
the ODD discipline kept intact. Read [docs/identity.md](docs/identity.md) before
any naming or branding change.

## Development loop

```bash
pnpm install            # hoisted node_modules; plain npm install at the root is unsupported
pnpm test               # unit tests, provider-contract mirror check, runtime harness
pnpm run typecheck
```

Run the suite from a shell that is **not** itself a gentle-shell session. A
gentle-shell session exports `GENTLE_SHELL_CONFIG_HOME`, `GENTLE_PI_AGENT_HOME` and
`PI_CODING_AGENT_DIR`; tests that isolate `HOME` still honour those, so
`tests/agent-home.test.ts` compares against the ambient home and fails. To run from
inside one:

```bash
env -u GENTLE_SHELL_CONFIG_HOME -u GENTLE_PI_AGENT_HOME -u PI_CODING_AGENT_DIR \
    -u GENTLE_SHELL_CHILD_PACKAGE_INJECTION -u GENTLE_SHELL_USER_PI_HOME pnpm test
```

## How work is tracked

Substantial work is tracked as an ODD feature document under
`odd/tasks/<feature>.md`: a short header, `## Specs` with the requirement text
quoted verbatim, `## Tasks` (ID, linked spec, route, commit) and `## Log`. The
document is the spec that implementers and reviewers read, so keep requirements
verbatim and append evidence to the Log instead of rewriting history.

## Pull requests

- One reviewable slice at a time, roughly 400 authored lines, tests and docs
  alongside the behavior they cover. Lock files do not count toward the slice.
- Conventional Commit messages (`feat:`, `fix:`, `docs:`, `test:`).
- A change whose risk touches data, auth, concurrency or delivery is its own commit
  so it can be reviewed alone.
- Code, comments, identifiers and docs are in English.

## Boundaries

- Do not rename the `gentle-ai` provider surface: it is an external contract
  (`docs/identity.md`, namespace boundary).
- Do not use upstream gentle-shell/gentle-pi names or logos as Atreus branding;
  truthful "based on gentle-shell" references are fine.
- This fork cannot publish the upstream npm package: `publish.yml` is gated to the
  upstream repository, and Atreus has no published package yet.
