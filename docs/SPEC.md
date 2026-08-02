# uilint — Technical Spec

> Status: Draft · 2026-08-02 · Spec version 1.0

## 1. Problem & Users

AI coding tools build the happy path. Ask one for a checkout form and you get a form that works
when the card clears, the network holds, and the list already has data. The loading state, the empty
state, the error state and the confirmation are absent — not because the model is bad at them, but
because nobody asked. The person building the app knows it too well to notice: they click the
buttons in the order that works. Users don't, and a silent failure reads to them as a broken product.

`uilint` is a Claude Code skill that refuses to let that ship. It carries a rule set about interface
*behaviour* — the states a screen must have, what makes an error message usable, how sections should
fail independently, and the named laws about how people actually read interfaces — and applies them
while UI is being written, not in a review that never gets scheduled.

**Primary user:** anyone building a UI with Claude Code who wants it to survive contact with a real
user — and who will not remember to ask for the error state themselves.
**Success looks like:** a component arrives with its loading, empty, error and success states already
present, and Claude names what is still missing rather than reporting "done".

## 2. Scope

### In scope

- Audit a component, screen or flow for **state completeness** (loading · empty · error · success · partial)
- Judge **error message quality** — what happened, why, what to do next — and flag silent failures
- Choose **error placement** (inline · toast · modal) from the severity of the failure
- Reduce **form friction** — validation timing, required-field clarity, prefill, forgiving input
- Enforce **graceful degradation** — one failing section must not blank the page
- Apply four **UX laws** (Jacob's, Hick's, Progressive Disclosure, Tesler's) to layout and flow decisions
- Act as a **blocking gate** during build: a component with a missing required state is not finished

### Explicitly out of scope

- **Visual craft** — spacing scales, colour, typography, elevation, gradients, focus-ring styling.
  The `craft` skill owns this and explicitly disclaims UX flow; duplicating it would mean two skills
  firing on the same prompt and disagreeing. `uilint` defers to it by name.
- **Accessibility auditing** — WCAG contrast, ARIA, screen-reader semantics. The `accessibility`
  skill owns it. Overlap is limited to focus/keyboard reachability, where `uilint` defers.
- **Heuristic evaluation as research output** — Nielsen-style scored audits belong to
  `ux-heuristics-review`. `uilint` produces code changes, not reports.
- **Automated static analysis** — no parser, no AST, no CI binary. v2 at the earliest; the rules are
  applied by a reading model, which is what lets them cover intent rather than syntax.
- **Design-time artifacts** — wireframes, user flows, personas. Other skills cover these.

## 3. Architecture

### Overview

A skill is a prompt-time artifact: a always-loaded instruction file plus reference files pulled in on
demand. The whole architecture question is therefore **what must be in context every time versus what
loads only when relevant**, because every always-loaded line is paid for on every unrelated request.

`SKILL.md` carries only the trigger description, the two modes, the five-state checklist, and the
output contract. The rules themselves live in four domain files, loaded when the work touches that
domain.

```
                        user prompt about UI
                                 │
                                 ▼
                 ┌───────────────────────────────┐
                 │ SKILL.md  (always loaded)     │
                 │  · triggers · modes           │
                 │  · 5-state checklist          │
                 │  · severity + output contract │
                 └───────────────┬───────────────┘
                    reads on demand │ (only the domains in play)
        ┌───────────────┬───────────┴───────┬──────────────────┐
        ▼               ▼                   ▼                  ▼
 references/     references/          references/        references/
  states.md      feedback.md            forms.md           laws.md
 loading·empty   errors·placement    validation·input   Jacob·Hick·
 ·partial        ·success feedback   ·prefill           disclosure·Tesler

        defers to ──►  craft (visual)  ·  accessibility (WCAG)
```

### Components

| Component | Responsibility | Technology |
| --- | --- | --- |
| `SKILL.md` | Trigger surface, mode selection, the 5-state checklist, severity semantics, output format. Does **not** contain individual rules. | Markdown + YAML frontmatter |
| `references/states.md` | Loading, empty, partial/degraded states; loader selection and timing thresholds | Markdown rule blocks |
| `references/feedback.md` | Error message quality, error placement, success confirmation | Markdown rule blocks |
| `references/forms.md` | Validation timing, required-field clarity, prefill, input tolerance | Markdown rule blocks |
| `references/laws.md` | Jacob's Law, Hick's Law, Progressive Disclosure, Tesler's Law — applied, not explained | Markdown rule blocks |
| `docs/SPEC.md` | This document. Ships in-repo so a fresh session can extend the skill correctly | Markdown |

### Decisions

**Rule storage** — plain Markdown rule blocks with a fixed heading shape, not YAML or JSON.
Because: the consumer is a language model reading prose, and Markdown keeps rules diffable and
readable on GitHub, which is where contributors will meet them.
Instead of: structured YAML — better for a future static analyser, worse for the only consumer that
exists today, and it would force a build step to render docs.
Revisit if: a real linter binary is built (out of scope, §2).

**Split references by domain, not one rules.md** — four files.
Because: a prompt about a form should not drag loading-spinner thresholds and Tesler's Law into
context. Splitting keeps per-request cost proportional to the work.
Instead of: a single `rules.md` — simpler, but every rule is paid for on every invocation.
Revisit if: total rules stay under ~15, where the split costs more than it saves.

**Severity drives enforcement** — every rule is `required` or `recommended`; only `required` blocks.
Because: a gate that blocks on everything gets disabled. Blocking is reserved for the failure modes
that silently harm users — missing error state, silent failure, non-actionable error.
Instead of: advisory-only — easy to scroll past, which is exactly how happy-path UIs ship today.
Revisit if: users report the gate firing on intentional omissions more than rarely.

**Compose with `craft`, don't absorb it** — `uilint` never comments on visual craft and says so.
Because: `craft` already owns the pixel level and its own description disclaims UX flow. Two skills
with clean edges both fire usefully; two skills with overlapping edges argue.
Instead of: one merged mega-skill — simpler install, but makes `craft` redundant and couples two
rule sets with different revision rates.
Revisit if: `craft` is retired.

**Rules are independently written directives, never source transcript text.**
Because: the rule set is derived from a creator's published series. Ideas and facts are free to
learn from; their expression is not. Rules are also *better* as compressed imperatives than as
transcribed speech.
Instead of: quoting the source — legally careless and worse as a rule file.
Revisit if: never.

## 4. Project Layout & Conventions

### Directory layout

```
uilint/
├── SKILL.md              # always loaded. Triggers, modes, checklist, output contract.
│                         # No individual rules — those live in references/.
├── references/           # loaded on demand. One file per rule domain, nothing else.
│   ├── states.md
│   ├── feedback.md
│   ├── forms.md
│   └── laws.md
├── docs/
│   └── SPEC.md           # this file. Design rationale only; never rules.
├── README.md             # for humans on GitHub: what it is, install, how to contribute rules
├── LICENSE               # MIT
└── .gitignore            # source material stays local
```

Nothing else at the root. No `src/`, no build step, no dependencies — a skill that needs a toolchain
to install will not be installed.

### Dependency direction

`SKILL.md → references/*.md`. One way. A reference file never points at `SKILL.md` and never at
another reference file — if two domains need the same rule, it belongs in the checklist in
`SKILL.md`, not cross-linked between references. Cross-links are how a rule set becomes unreadable.

### Rule ID naming

| Kind | Convention | Example |
| --- | --- | --- |
| Rule ID | `R-<domain>-<nn>` | `R-STATE-03` |
| Domain codes | `STATE` · `FEEDBACK` · `FORM` · `LAW` | — |
| Reference file | lowercase domain noun, plural | `states.md` |

IDs are permanent. A retired rule keeps its number and is marked retired; renumbering breaks every
review that ever cited it.

### Size limits

Enforced by review, because every line is context cost on real requests.

| Unit | Soft | Hard |
| --- | --- | --- |
| `SKILL.md` | 120 lines | 200 lines |
| One reference file | 150 lines | 250 lines |
| One rule block | 6 lines | 10 lines |
| Total rules | 30 | 40 |

Hitting the hard limit means merging or cutting rules, never raising the limit.

### Tooling

| Concern | Tool |
| --- | --- |
| Format | Markdown, 100-col soft wrap |
| Validation | `SKILL.md` frontmatter must parse as YAML with `name` and `description` |
| CI | None at v1. A link-check and frontmatter-parse action when the repo goes public |

## 5. Data Models

A rule is the unit of everything here. Fixed shape, so rules stay comparable and a reviewer can cite
one precisely.

```markdown
### R-STATE-03 · Error state per data section
**DO:** Give every section that fetches data its own error state and its own retry control.
*Why:* One failed request should cost the user that section, not the page.
*Applies:* any component performing an async fetch
*Check:* force the request to reject — is the rest of the page still usable?
*Severity:* required
```

| Field | Type | Notes |
| --- | --- | --- |
| Heading | `R-<DOMAIN>-<nn> · <short name>` | Permanent ID; short name may be reworded |
| `DO` \| `DON'T` | one imperative sentence | Exactly one of the two, never both for one idea |
| `*Why:*` | one line | Mandatory. A rule that cannot state why it exists cannot state when to break it |
| `*Applies:*` | scope predicate | What must be true for the rule to be in play |
| `*Check:*` | observable test | How to verify — must be executable by a person in under a minute |
| `*Severity:*` | `required` \| `recommended` | `required` blocks the gate; `recommended` advises |

**Constraints**

- `Why` present on every rule — enforced by review; a rule without one is deleted, not fixed later.
- `Severity: required` only where absence causes silent user harm: no feedback at all, a
  non-actionable error, an unrecoverable dead end. Everything else is `recommended`.
- Numbers appear only where they can be stated with confidence. A vague statement is never laundered
  into false precision — where a figure cannot be stood behind, the rule is written to rest on its
  reasoning instead, or the number is dropped. Rule files carry no inline editorial notes.
- A rule contradicting an existing rule is added with `<!-- CONFLICT: ... -->` for a human to
  resolve, never silently overwriting.

## 6. Interfaces

### Trigger surface

The `description` field is the only thing Claude matches on when deciding to load the skill, so it is
an interface, not documentation. It must name the artefacts (component, screen, form, flow) and the
symptoms ("nothing happens when I click", "it just spins") — not only the jargon, because users
describe the bug, not the category.

### Invocation

| Path | Trigger | Mode |
| --- | --- | --- |
| Automatic | User is writing or editing UI that fetches data, submits, or navigates | Build gate |
| Automatic | User asks why an interface feels broken, confusing, or unfinished | Review |
| Explicit | `/uilint` | Review of whatever is in context |

### Output contract — review mode

```
<one-line verdict>

Blocking — <n> required state(s) missing:
  <file>:<line> · R-<ID>
    <what is absent, in terms of what the user experiences>
    Fix: <concrete change>

Worth fixing:
  <file>:<line> · R-<ID> — <one line>

<single offer to apply the fixes>
```

Rules: only violated rules appear. A clean component gets the verdict line and nothing else. No
preamble, no restatement, no score.

### Output contract — build gate

When a component is written or edited and a `required` rule is unmet, Claude states plainly that the
component is not finished, names the missing state in terms of user consequence ("a failed payment
currently shows nothing at all"), and offers to add it. It does not silently add states the user
didn't ask for, and it does not report the work complete.

### Composition with other skills

| Situation | Behaviour |
| --- | --- |
| Visual craft issue noticed (spacing, colour, gradient) | Do not report. Name `craft` once if it is clearly relevant |
| Contrast / ARIA / screen-reader issue | Do not report. Name `accessibility` once |
| User explicitly wants a full heuristic audit | Name `ux-heuristics-review` |

## 7. Core Flows

### Flow A — Build gate (the primary flow)

1. User asks Claude to build a component that fetches, submits, or navigates.
2. Skill loads → `SKILL.md` five-state checklist enters context.
3. Claude writes the component, applying `required` rules as it goes rather than afterwards.
4. Before reporting completion, Claude walks the checklist against what it just wrote.
5. Any unmet `required` rule → states the component is unfinished, names the gap by user
   consequence, offers the fix.

**Failure branches:** user says "just the happy path for now" → acknowledge once, record it, stop
raising it for that component. The gate informs; it does not nag.

### Flow B — Review existing code

1. User points at a file, or asks why something feels off.
2. Claude reads the relevant reference files — only the domains in play.
3. Each rule checked against the actual code, not against memory of the rule set.
4. Findings anchored to `file:line`, sorted blocking-first.
5. Output per §6. Clean file → one line.

**Failure branches:** no UI in context → say so and ask what to look at, rather than reviewing
something adjacent.

### Flow C — Screen audit

1. User asks for a whole-screen or whole-flow check.
2. Claude enumerates each data-bearing section.
3. Produces a coverage matrix: section × (loading · empty · error · success), each cell present or
   absent with the file that would need to change.
4. Blocking items summarised beneath.

**Failure branches:** screenshot-only input → state that state coverage cannot be judged from a
single rendered frame, and audit only what is visible.

## 8. Edge Cases & Failure Modes

| Case | Consequence if unhandled | Handling |
| --- | --- | --- |
| Non-visual code in context (CLI, library, migration) | Skill fires on irrelevant work and wastes context | `Applies:` predicate on every rule; skill exits silently when nothing matches |
| Component legitimately has no empty state (always ≥1 item) | False positive erodes trust in the gate | `Applies:` requires a *collection that can be empty*; single-record views are out of play |
| Internal tool where the user accepts rough edges | Gate becomes noise; user disables skill | Honour an explicit opt-out for the session, once, without re-raising |
| Screenshot with no code | Rules about retry logic are unverifiable from pixels | Report those as "verify in code", never as violations |
| Framework unknown or unusual | Fixes suggested in the wrong idiom | Fixes described behaviourally when the framework is unclear; concrete code only when it is known |
| Two rules disagree | Reviewer cites both, user loses confidence | `<!-- CONFLICT -->` marker; a human resolves before merge |
| Rule set outgrows context budget | Every unrelated prompt pays for it | Hard limits, §4. At the limit rules merge or die |
| Source series continues past the captured parts | Rule set silently goes stale | Open question, §12 — a documented refresh path, not an assumption that it is complete |

## 9. Security & Permissions

**Authentication / authorization:** none. A skill is a set of Markdown files read by the model; there
is no runtime, no network access, no state.

**The one real security rule** is a rule *in* the set rather than a property of it: error messages
must never surface backend or database detail to end users. A raw exception string is both
unreadable and a disclosure of internal structure. This is `R-FEEDBACK-01`, severity `required`, and
it is the only rule that is a security control as well as a UX one.

**Data handling:** the repo contains no source material. Transcripts, audio, and any downloaded media
stay local and gitignored. Only independently written rules are committed. Creators whose published
work informed the rule set are credited in `README.md`.

## 10. Build Order

**M1 — The gate works**
Claude refuses to call a fetching component finished while its error state is missing.
- [ ] Rewrite `SKILL.md`: description aimed at behaviour symptoms, two modes, five-state checklist, severity semantics, output contract
- [ ] Write `references/states.md` — loading, empty, partial/degraded, loader selection and timing
- [ ] Delete the visual-craft framing and the placeholder `references/rules.md`
- [ ] Verify on a real component with a deliberately missing error state

**M2 — Feedback quality**
A vague error message gets flagged and rewritten.
- [ ] `references/feedback.md` — error content, error placement, success confirmation

**M3 — Forms**
- [ ] `references/forms.md` — validation timing, required clarity, prefill, input tolerance

**M4 — Laws**
- [ ] `references/laws.md` — Jacob's, Hick's, Progressive Disclosure, Tesler's, each stated as an applied rule rather than an explanation

**M5 — Public**
- [ ] README: install, usage, contribution bar, credits
- [ ] Push `DahanItamar/uilint`
- [ ] Confirm the junction at `~/.claude/skills/uilint` still resolves after publish

## 11. Assumptions

1. **The rule set is derived from one creator's series and credited as such.** If material from other
   sources is added later, `README.md` credits grow and no rule text changes.
2. **Rules are synthesised, not transcribed** — per your "not hardcoded" instruction. Each rule is an
   independently written imperative; none reproduces source wording.
3. **"Pick's Law" in the source is Hick's Law** and is written under the correct name. If the source
   genuinely meant something else, `references/laws.md` changes; nothing else does.
4. **Five states, not four.** The source names four (loading, empty, error, success); partial /
   degraded is treated as a first-class fifth because two full parts of the series are devoted to it.
   If this proves noisy, partial folds back into `states.md` as a sub-rule.
5. **The junction at `~/.claude/skills/uilint` is the install mechanism for you**, so repo edits are
   live without copying. Other users copy the folder.
6. **No CI at v1.** If the repo attracts contributors, a frontmatter-parse and link-check action is
   the first addition.
7. **Loader-selection rules are written from Parts 2 and 4 without Part 3.** Those two parts already
   give the shape — skeletons for whole-layout waits, spinners for buttons and small regions, and
   explicit timing thresholds — so the rules stand on that. Where Part 3 would have supplied a
   figure, the rule is phrased to rest on its reasoning rather than on a number nobody verified. If
   Part 3 later contradicts them, only `references/states.md` changes.

## 12. Open Questions

- **Part 3 of the source series was never captured.** Decided rather than left open — see Assumption
  7; M1 is not blocked. Capturing it later would refine the loader-selection rules, not invalidate
  them. — blocks: nothing · needed by: whenever the transcript is available
- **The series is still running** (Part 16 points at Part 17). No refresh path is defined for folding
  later parts into the rule set. — blocks: nothing today · needed by: M5
- **Does the gate hold for prototypes?** A user who says "quick mockup" arguably wants the gate off by
  default rather than per-component. — blocks: nothing · needed by: post-launch, from real usage
