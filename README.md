<div align="center">

# uilint

**A linter for the states everyone forgets — it refuses to call a component finished while its
error state is missing, and stays switched on because it blocks on only 16 of its own 39 rules.**

A Claude Code skill: 39 rules about interface behaviour, applied while the UI is written.
**No dependencies, no build step, nothing to run.**

<img alt="version 1.1.0" src="https://img.shields.io/badge/version-1.1.0-B55400?style=flat-square">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-2b2e3a?style=flat-square"></a>
<img alt="zero dependencies" src="https://img.shields.io/badge/dependencies-0-1f6f3f?style=flat-square">
<img alt="39 rules" src="https://img.shields.io/badge/rules-39-2b2e3a?style=flat-square">
<img alt="16 rules block completion, 23 report only" src="https://img.shields.io/badge/blocking-16%20of%2039-1f6f3f?style=flat-square">

<a href="SKILL.md">Skill</a> ·
<a href="docs/SPEC.md">Spec</a> ·
<a href="references/states.md">Rules</a>

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

## The gate that stays on

Most quality gates die the same way: they block on everything, someone has a deadline, and the gate
gets switched off permanently. So severity here is rationed. **16 rules block completion. The other
23 are reported and never block.**

```mermaid
flowchart TD
    C["A section that fetches,<br/>submits, or navigates"] --> W["Walk the five states"]
    W --> F{"Gap found?"}

    F -- "all five handled" --> DONE["Finished"]
    F -- "gap" --> S{"Which severity?"}

    S -- "required · 16 rules<br/><i>silent user harm</i>" --> BLOCK["Blocks completion"]
    S -- "recommended · 23 rules<br/><i>everything else</i>" --> REPORT["Reported, never blocks"]

    REPORT --> DONE
    BLOCK --> FIX["Fix, then re-check"]
    FIX --> W
```

`required` is reserved for **silent user harm**: no feedback at all, an error nobody can act on, a
dead end with no way forward, internal detail leaked on screen. Everything else reports. That ratio
is the design — a gate trusted enough to leave on is worth more than a stricter one that gets
disabled in week two.

Findings lead with what the user experiences, not which rule fired. *"A declined card currently
shows nothing at all"* lands; *"violates R-FEEDBACK-02"* does not.

**Every rule carries a check you can run in a minute.** All 39 of them — verified by counting: 39
rules, 39 `Why` lines, 39 `Applies` predicates, 39 `Check` steps, no exceptions.

```markdown
### R-STATE-12 · One failure must not blank the page
**DON'T:** Replace an entire screen with an error because a single section failed.
*Why:* The rest of the data arrived and is still useful. Discarding it turns a
partial outage into a total one.
*Applies:* screens with more than one data region
*Check:* force one request to reject. Is the remaining content still readable?
*Severity:* required
```

The `Applies` line is what stops false positives — it is the predicate that keeps a rule about
multi-region screens from firing on a single-panel one.

## A real run

Pointed at the front-end of a working local tool — a 1,000-line vanilla-JS page that polls a Python
server every five seconds:

```text
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
code is *correct*, the happy path is fine, and the failure is invisible by construction. **A
`catch` that swallows an error is a decision to tell the user nothing.**

## Install

This repository is its own marketplace — nothing else to add first.

```bash
/plugin marketplace add DahanItamar/uilint
/plugin install uilint@uilint          # invokes as /uilint:uilint
```

Or clone it straight into your skills directory, which needs no plugin UI and so works in web and
cloud sessions too:

```bash
git clone https://github.com/DahanItamar/uilint.git ~/.claude/skills/uilint
```

```powershell
git clone https://github.com/DahanItamar/uilint.git "$env:USERPROFILE\.claude\skills\uilint"
```

Take the clone route if you want to edit the rules — a copy you own beats a cache you don't. Pick
one route, not both, or the same rules load twice. Restart Claude Code or run `/reload-plugins`
afterwards.

> [!IMPORTANT]
> **In the VS Code extension the command is `/plugins`, plural, and it opens a dialog.** The
> `/plugin` lines above are terminal-CLI syntax and do nothing there — they fail silently, which
> looks identical to the install not working. In the extension: `/plugins` → **Marketplaces** → add
> `DahanItamar/uilint` → **Plugins** → **uilint** → **Install**.

It also triggers on its own while you are building UI that fetches, submits, or navigates — or when
you describe a symptom rather than a category: *"nothing happens when I click"*, *"it just spins
forever"*, *"users don't know if it worked"*.

## Under the hood — briefly

- **Two files deep, on purpose.** [`SKILL.md`](SKILL.md) is 103 lines holding only the trigger, the
  checklist and the output contract. The 39 rules live in four reference files totalling 348 lines,
  loaded on demand — so a question about a form does not drag spinner thresholds and Tesler's Law
  into context.
- **Four domains.** [`states.md`](references/states.md) 14 · [`feedback.md`](references/feedback.md)
  9 · [`forms.md`](references/forms.md) 8 · [`laws.md`](references/laws.md) 8 — Jacob's, Hick's,
  Tesler's, and progressive disclosure.
- **Fixed rule shape.** DO or DON'T, never both; a mandatory *Why*; an `Applies` predicate; a
  runnable `Check`. Enforced across all 39.
- **Permanent IDs.** A retired rule keeps its number, so a review citing `R-FORM-03` in 2026 still
  means something in 2028.
- **39 rules against a hard maximum of 40.** One slot left, deliberately. New rules merge with or
  replace existing ones rather than accumulating, because every line is context cost on every
  unrelated prompt.
- **Composes rather than competes.** It names `craft` for visual issues and `accessibility` for
  contrast and ARIA, and comments on neither itself.
- **Portable to any agent.** The plugin format is Claude Code's, but `SKILL.md` is plain Markdown
  with nothing to run — paste its body into `AGENTS.md`, a Cursor rule, or a system prompt. What you
  lose is automatic invocation.

> It deliberately does not touch spacing, colour, typography, elevation, or motion — those belong to
> the `craft` skill, whose own description disclaims UX flow. The two were built to leave each other
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

Built by <a href="https://github.com/DahanItamar">Itamar Dahan</a> · <a href="LICENSE">MIT</a> · © 2026

</div>
