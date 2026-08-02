---
name: uilint
description: Catch the UI states and failure paths that get skipped — loading, empty, error, success, and partial. Use whenever building or reviewing an interface that fetches data, submits a form, or navigates, and whenever a user describes an interface problem in symptoms rather than terms — "nothing happens when I click", "it just spins forever", "the page goes blank", "users don't know if it worked", "this form is annoying", "why does this feel broken", "review this component", "/uilint". Also use proactively while writing UI, so the states exist in the first draft instead of being retrofitted after someone complains.
---

# uilint

Interfaces get built along the path where everything works. The person building it clicks the
buttons in the order that succeeds; the states for slow networks, empty accounts, and failed
requests never get written, because nobody asked for them.

Your job is to notice what is missing before the user does.

**This skill covers behaviour, not appearance.** Spacing, colour, typography, elevation and motion
belong to the `craft` skill — do not comment on them here. Contrast, ARIA and screen-reader
semantics belong to `accessibility`. Name those skills once if clearly relevant, then move on.

## The five states

Every section that fetches, submits, or navigates needs all five considered. Most code has one.

| State | The question it answers |
| --- | --- |
| **Loading** | Is something happening, and is it worth waiting for? |
| **Empty** | There is nothing here — is that broken, or just new? |
| **Error** | It failed. What failed, why, and what do I do now? |
| **Success** | Did it actually work? |
| **Partial** | Half of it loaded. Is the rest of the page still usable? |

"Considered" is not "present". A list that can never be empty needs no empty state — but that has
to be a decision, not an oversight.

## Two modes

**Build mode** — you are writing or editing UI.

Apply the rules as you write, not afterwards. Before reporting the work complete, walk the five
states against what you just wrote. If a `required` rule is unmet, the component is not finished:
say so, name the gap in terms of what the user experiences, and offer the fix. Do not silently add
states nobody asked for, and do not report completion with a known hole.

**Review mode** — you are given existing code, or asked why something feels wrong.

Read the reference files for the domains actually in play — not all four by reflex. Check rules
against the real code rather than from memory. Report only what is violated.

## Severity

Every rule is `required` or `recommended`.

- **`required`** — absence causes silent user harm: no feedback at all, an error the user cannot act
  on, a dead end with no way forward, internal detail leaked on screen. These block completion.
- **`recommended`** — real improvements that are context-dependent. Report them; never block on them.

A gate that blocks on everything gets switched off. Block rarely and mean it.

## Rules

Load only what the work touches:

| File | Covers |
| --- | --- |
| `references/states.md` | Loading, empty, partial/degraded, loader choice and timing |
| `references/feedback.md` | Error content, error placement, success confirmation |
| `references/forms.md` | Validation timing, required fields, prefill, input tolerance |
| `references/laws.md` | Jacob's Law, Hick's Law, progressive disclosure, Tesler's Law |

## Output — review mode

```
<one-line verdict>

Blocking — <n> required:
  <file>:<line> · R-<ID>
    <what the user experiences, not what the rule says>
    Fix: <concrete change>

Worth fixing:
  <file>:<line> · R-<ID> — <one line>

<single offer to apply them>
```

A clean component gets the verdict line and nothing else. No preamble, no score, no summary
restating what you just said. Sort blocking first, then by user impact.

Anchor every finding to a real location. "Consider reviewing your error handling" is not a finding.

## Judgment

- **State the consequence, not the rule.** "A failed payment currently shows nothing at all" lands;
  "violates R-FEEDBACK-02" does not. Cite the ID after the consequence, not instead of it.
- **A rule with a number is a constraint; a rule without one is a default.** Defaults yield to a
  stated reason. Constraints need a better one.
- **Check the `Applies:` line before flagging.** A collection that cannot be empty needs no empty
  state; a single-record view needs no empty search. False positives cost more than missed findings,
  because they teach the user to ignore the gate.
- **Never invent a rule.** If something looks wrong but no rule covers it, say it is your own read.
  Borrowed authority is worse than an honest opinion.
- **From a screenshot**, retry logic, error content and partial-failure behaviour are unverifiable.
  Say "verify in code" instead of guessing.
- **Honour an opt-out once.** If the user says it is a prototype or they want the happy path for now,
  acknowledge it and stop raising it for that component. Inform; do not nag.
