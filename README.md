# public-client-leak-audit

A [Claude](https://claude.com/claude-code) **agent skill** for auditing a public-facing client so its public surface stays *self-contained* — revealing the calls it makes and the data it exchanges, but nothing about how the backend works.

Any client that ships to users **and** talks to a private backend — a browser extension, a mobile or desktop app, a single-page web app, a CLI, an SDK, an open-sourced client — inevitably exposes its own source. The risk is everything *beyond* the unavoidable: comments explaining server logic, disclosed rate limits and anti-abuse mechanics, backend tech-stack and infra names, private repo paths, test scaffolding that documents server behavior, and secrets sitting one bundle-step from publication. Each of those is free reconnaissance for an attacker and a lever for abuse.

This skill turns a real audit into a repeatable method: **scope the public/private boundary → sweep for leaks → harden the client → remediate → verify → report.**

## What it checks

- **Disclosure leaks** — private repo/paths, backend stack & infra identifiers, anti-abuse/rate-limit/quota mechanics, auth & account internals, secrets (in-tree and one-step-from-publication), test/e2e code that encodes backend behavior, server-explaining comments, dead code hardcoding server policy, and docs/commit messages describing backend architecture.
- **Client-side hardening** — permission/scope minimization, IPC/message sender & origin validation, auth-token storage and egress, DOM/XSS sinks, network-layer origin pinning, build-time config gating (so a production build can't be repointed or bundle a secret), source-bundle hygiene, remote-config/code validation, and a supply-chain quick pass.
- **API over-disclosure** — separating *necessary-minimum* (endpoints, payload shapes, `status → UI` mappings) from *over-disclosure* (server-side processing, limits the client doesn't need, endpoints it never calls).

## The core idea

> A public client may show the calls it makes and the data shapes it exchanges — but nothing beyond that.

And the **rewrite rule** it applies to comments, strings, and docs:

> Client code may state the *contract* (what input produces what observable result). It must not explain *what or why the server does* it.

## Files

```
SKILL.md                        # the method Claude follows (loaded when the skill triggers)
references/
  leak-taxonomy.md              # what to hunt, why it matters, starter search patterns
  rewrite-rules.md              # the comment/string rewrite rule + before/after examples
  client-hardening.md           # client-side security checklist
  report-template.md            # output report structure
scripts/
  leak-sweep.sh                 # parameterized ripgrep sweep to seed the audit
```

`SKILL.md` stays lean; the `references/` files are read on demand. `scripts/leak-sweep.sh` runs a first pass — customize its `PRODUCT_TERMS` with the product's own private vocabulary.

## Install

Claude Code discovers skills under a `skills/` directory. Clone or copy this repo so the skill folder lands there:

```bash
# personal (all projects)
git clone <this-repo> ~/.claude/skills/public-client-leak-audit

# or per-project
git clone <this-repo> .claude/skills/public-client-leak-audit
```

The directory must contain `SKILL.md` at its top level. Restart Claude Code (or reload skills) so it picks up the new one.

## Use

Just ask, in your own words — the skill triggers on intent, not a magic command:

- "Audit this repo for anything that leaks backend details before we open-source it."
- "Does the extension disclose our rate limits or server internals?"
- "Is this client safe to publish? Check secrets and comments."
- "Security review of the client — permissions, message handlers, token handling."

Claude scopes the public/private boundary first (it may ask what your private backend/stack is — that's what makes the sweep precise), runs the sweep, reads hits in context, applies fixes with the rewrite rule, verifies against your build/tests, and hands back a severity-ordered report with residual recommendations (secret rotation, git-history exposure, deferred tradeoffs).

You can also run the sweep standalone:

```bash
scripts/leak-sweep.sh path/to/public/client
```

## Scope & intent

Defensive by design: it reduces a client's public attack surface and hardens it. It does not build exploits. Run it on code you own or are authorized to audit.

## License

Add a license of your choice when you create the repository (e.g. MIT or Apache-2.0).
