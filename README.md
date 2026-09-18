# AI working agreements

Three configs for AI coding assistants, each kept in both `CLAUDE.md` and
`AGENTS.md` form so it works with whichever tool a repo's contributors are
on.

They're written as instructions to the model, not documentation about the
tooling. That distinction matters more than it sounds like it does — a
config that describes what an agent does gives the agent nothing to act
on.

## What's here

**`/CLAUDE.md`** — a general, project-agnostic working agreement for
day-to-day AI-assisted coding. Non-negotiables, the autonomy split, and
human-in-the-loop gates. Anything repo-specific belongs in a `PROJECT.md`
next to it.

**`/code-reviewer-agent/`** — a review config for a multi-tenant Node /
TypeScript backend. Reads the repo's architecture doc for context,
enforces a core-class boundary, and posts inline comments on the diff. It
comments; it doesn't approve, merge, or push.

**`/oncall-debugger-agent/`** — on-call triage, triggered from a chat
slash command. Queries an APM, a log store, and a cloud provider in
parallel, correlates the three, and posts one diagnosis with evidence back
into the thread. Strictly read-only — it surfaces a hypothesis, it never
remediates.

## Shared philosophy

**Specific, checkable rules instead of vague best practices.** "Write
clean code" doesn't change what gets generated. "No `await` inside a loop
over more than ten items" does.

**A hard line between what the AI does alone and what needs a human,**
stated explicitly rather than left to judgement in the moment.

**Every threshold is a number.** Correlation windows, comment caps, speed
budgets, loop sizes. A rule that justifies itself without stating the
number it depends on is the same rule as no rule.

**Noise is a failure mode.** Too many flags and people stop reading them,
which is worse than not flagging at all. Both agent configs cap their own
output for that reason.

**Read-only is a design position, not a missing feature.** The on-call
agent could restart a service. It doesn't, because a wrong remediation
during an incident costs more than a slow one.

## Precedence

The root working agreement applies everywhere. Where an agent config is
more specific, the agent config wins. Where they conflict outright, the
agent says so rather than picking one silently.

## Adapt before use

These are working configs rather than templates, so a few things are
concrete where yours will differ:

- The `Base*` core-class prefix and the `src/tenants/<tenantId>/`
  directory in the reviewer. Swap for your own convention. What matters is
  that the boundary is named and mechanically detectable.
- The architecture doc path the reviewer loads before it starts.
- The 5 and 15 minute correlation windows in the on-call agent. Tune to
  your own consumer lag.
- The 90-second speed budget, the 15-comment cap, the 10-line log excerpt
  limit.
- The PII list in the redaction rule. Add whatever your domain carries
  that shouldn't reach a chat channel.

## Keeping the two file names in sync

`AGENTS.md` ships as a byte-identical copy of `CLAUDE.md` in each
directory. Neither file's heading names a filename, so the same content is
correct under either name.

Edit `CLAUDE.md` only, and generate the other:

    for d in . code-reviewer-agent oncall-debugger-agent; do
      cp "$d/CLAUDE.md" "$d/AGENTS.md"
    done

Put that in a pre-commit hook, or replace both with symlinks
(`ln -s CLAUDE.md AGENTS.md`) if your tooling follows them. What matters is
that you don't maintain two copies by hand — they will drift, and you
won't notice which one the tool actually read.
