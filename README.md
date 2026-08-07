<div align="center">

# uilint

**A linter for the states everyone forgets — it refuses to call a component finished while its error state is missing.**

A Claude Code skill: 39 rules about interface behaviour, applied while the UI is written. **No dependencies, no build step, nothing to run.**

</div>

---

## The five states

Ask an AI tool for a checkout form and you get one that works when the card clears, the network
holds, and the list already has data. The person building it clicks the buttons in the order that
succeeds, so the other paths never get written.

`uilint` walks five states against every section that fetches, submits, or navigates:

| State | The question it answers |
|---|---|
| **Loading** | Is something happening, and is it worth waiting for? |
| **Empty** | There is nothing here — is that broken, or just new? |
| **Error** | It failed. What failed, why, and what do I do now? |
| **Success** | Did it actually work? |
| **Partial** | Half of it loaded. Is the rest of the page still usable? |

Considered is not the same as present. A list that can never be empty needs no empty state — but
that has to be a decision, not an oversight.

## A real run

Pointed at the front-end of a working local tool — a 1,000-line vanilla-JS page that polls a Python
server every five seconds:

```
Two required rules broken, both silent.

  web/index.html:1019 · R-FEEDBACK-02
    The five-second poll swallows every failure: refresh().catch(() => {}).
    Kill the server and the page keeps showing stale data with no indication
    it stopped updating. The user is reading numbers that stopped being true.
    Fix: surface a stale/disconnected marker after two consecutive failures

  web/index.html:917 · R-FEEDBACK-02
    es.onerror closes and reconnects the event stream silently, so a dropped
    connection is invisible until someone notices progress has stopped.
    Fix: show a reconnecting state while the retry loop runs

Worth fixing:

  web/index.html:1012 · R-STATE-02
    First paint renders every panel empty before /api/state resolves — a new
    user cannot tell an empty library from one that has not loaded yet.
```

Both blocking findings are the same failure mode, and it is the one this rule set exists for: the
code is *correct*, the happy path is fine, and the failure is invisible by construction. A `catch`
that swallows an error is a decision to tell the user nothing.

## The gate

Every rule is `required` or `recommended` — **16 and 23** respectively.

`required` is reserved for silent user harm: no feedback at all, an error nobody can act on, a dead
end with no way forward, internal detail leaked on screen. Those block completion. Everything else
is reported and never blocks, because a gate that blocks on everything gets switched off.

Findings lead with what the user experiences, not which rule fired. "A declined card currently shows
nothing at all" lands; "violates R-FEEDBACK-02" does not.

Every rule carries a one-minute test you can actually run:

```markdown
### R-STATE-12 · One failure must not blank the page
**DON'T:** Replace an entire screen with an error because a single section failed.
*Why:* The rest of the data arrived and is still useful. Discarding it turns a
partial outage into a total one.
*Applies:* screens with more than one data region
*Check:* force one request to reject. Is the remaining content still readable?
*Severity:* required
```

## Install

Two routes. Pick one — installing both loads the same rules twice.

### As a plugin — updates itself

```
/plugin marketplace add DahanItamar/ai-skills
/plugin install uilint@dahanitamar
```

Invokes as `/uilint:uilint` (plugin skills are namespaced). Update with
`/plugin marketplace update dahanitamar`. The catalogue also carries
[`readme-architect`](https://github.com/DahanItamar/readme-architect) and
[`flowsystem`](https://github.com/DahanItamar/flowsystem) — one add, install what you want.

### As a skill — editable

```bash
git clone https://github.com/DahanItamar/uilint.git ~/.claude/skills/uilint
# Windows: git clone https://github.com/DahanItamar/uilint.git "%USERPROFILE%\.claude\skills\uilint"
```

Invokes as `/uilint`. Update with `git pull`. Prefer this if you want to change the rules — a clone
you own beats a cached copy you don't.

### Any other agent

The plugin format is Claude Code's; Codex, Cursor and the rest do not read it. But
[`SKILL.md`](SKILL.md) is plain Markdown with no code and nothing to run — paste its body into
`AGENTS.md`, a Cursor rule, or a system prompt and it works. What you lose is automatic invocation:
Claude Code loads it when it becomes relevant, other tools need you to point at it.

Either way it triggers on its own whenever you are building UI that fetches, submits, or navigates —
or when you describe a symptom rather than a category: *"nothing happens when I click"*, *"it just
spins forever"*, *"users don't know if it worked"*.

Nothing to configure or keep running. It is Markdown.

## Under the Hood — Briefly

- **Two files deep, on purpose** — [`SKILL.md`](SKILL.md) is 103 lines and holds only the trigger,
  the checklist and the output contract. The rules live in four reference files loaded on demand, so
  a question about a form does not drag spinner thresholds and Tesler's Law into context.
- **Rule domains** — [`states.md`](references/states.md) (14): loading, empty, partial, loader
  choice and timing · [`feedback.md`](references/feedback.md) (9): error content, placement, success
  confirmation · [`forms.md`](references/forms.md) (8): validation timing, required fields, input
  tolerance · [`laws.md`](references/laws.md) (8): Jacob's, Hick's, progressive disclosure, Tesler's.
- **Fixed rule shape** — DO or DON'T, never both; a mandatory *Why*; an `Applies` predicate that
  prevents false positives on components the rule was never meant for; a runnable `Check`.
- **Permanent IDs** — a retired rule keeps its number, so a review that cited `R-FORM-03` in 2026
  still means something in 2028.
- **A hard ceiling of 40 rules.** At 39 the set is full. New rules merge with or replace existing
  ones rather than accumulating, because every line is context cost on every unrelated prompt.
- **Composes rather than competes** — it names `craft` for visual issues and `accessibility` for
  contrast and ARIA, and comments on neither itself.

> It deliberately does not touch spacing, colour, typography, elevation, or motion. Those belong to
> the `craft` skill, whose own description disclaims UX flow — the two were built to leave each other
> alone. Design rationale is in [`docs/SPEC.md`](docs/SPEC.md).

## Credits

The behavioural rules were distilled from
**[@synsation_](https://www.instagram.com/synsation_/)**'s *Build for Good UX* series — a genuinely
practical walkthrough of the states and failure paths most tutorials skip. If these rules are useful
to you, the series is worth your time.

They are independently written directives derived from that material; no transcripts or source text
appear in this repository. Jacob's Law, Hick's Law and Tesler's Law are long-established principles,
credited to Jakob Nielsen, William Hick and Larry Tesler.

---

<div align="center">

Built by <a href="https://github.com/DahanItamar">Itamar Dahan</a> · MIT · © 2026

</div>
