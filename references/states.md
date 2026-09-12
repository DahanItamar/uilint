# States

Loading, empty, and partial. Error content lives in `feedback.md`; this file covers whether a state
exists at all and which form it takes.

`R-STATE-03` and `R-STATE-05` are retired — their material was merged into `R-STATE-06` and
`R-STATE-04`. Retired IDs are never reused, so a review citing one still resolves.

---

### R-STATE-01 · All five states considered
**DO:** Decide, for every section that fetches or submits, what it shows while loading, when it has
nothing, when it fails, when it succeeds, and when only part of it is ready.
*Why:* The path where everything works is the one that gets built by default; the other four are what
users actually hit.
*Applies:* any component performing async work
*Check:* name the file and line for each of the five. A state you cannot point at does not exist.
*Severity:* required

### R-STATE-02 · Never show an unexplained blank
**DO:** Render an indicator as soon as a wait begins.
*Why:* A blank screen is indistinguishable from a broken one, and people give up on it within a few
seconds.
*Applies:* any wait with no existing content on screen
*Check:* throttle the network to slow 3G and load the page. Is anything on screen immediately?
*Severity:* required

### R-STATE-04 · Long waits need words, then a measure
**DO:** Put text on a wait that runs past a few seconds and advance it as the work moves; past ~10
seconds, switch to determinate progress or an explicit step list.
*Why:* An indefinite animation with no words reads as stuck, while text that advances shows the
system still working — people wait substantially longer for it. But a loop that never resolves
inverts patience, because the user cannot tell progress from a hang.
*Applies:* multi-step or network-bound operations — uploads, imports, builds, batch jobs
*Check:* past ~5 seconds, does the screen say what is happening? Can it exceed ~10 seconds? Then it
needs progress, not a spinner.
*Severity:* recommended

### R-STATE-06 · Match the loader to the wait
**DO:** Fit the affordance to the wait — nothing under ~1 second, a skeleton mirroring the layout
when a whole region is arriving, an inline spinner in the control when one action is in flight.
*Why:* A spinner that appears and vanishes inside a second registers as a glitch, not as progress.
Past that, a skeleton lets the eye settle into the layout; a skeleton inside a button is noise, and
a page-wide spinner throws away the layout information you already have.
*Applies:* any loading affordance, at any duration
*Check:* time it first — under a second, render the result directly. Otherwise, does the loader
occupy the same footprint the real content will?
*Severity:* recommended

### R-STATE-07 · Fail as soon as you know
**DON'T:** Run a loader for its full duration when the outcome is already known to be a failure.
*Why:* Making someone wait and then telling them it failed spends their patience to deliver bad
news, which reads as contempt for their time.
*Applies:* validation that can fail early, requests that reject fast
*Check:* force a failure. Does the error appear when it is known, or when the timer ends?
*Severity:* required

### R-STATE-08 · An empty state must say what it is for
**DO:** Explain what belongs in an empty region and give the primary action to fill it.
*Why:* This is frequently the first screen a new user sees, and a blank panel with no next step reads
as a product that is broken rather than one that is new.
*Applies:* any collection that can legitimately hold zero items
*Check:* sign up as a new user. Is there an obvious next action on every empty region?
*Severity:* required

### R-STATE-09 · Empty results keep a way forward
**DO:** Offer a route out of a no-results state — relax a filter, search a broader term, clear the query.
*Why:* A dead end costs you the session; the user came with intent and you are the one holding the
information about what else exists.
*Applies:* search, filtering, faceted browse
*Check:* search for something with no matches. Is there anything to click?
*Severity:* recommended

### R-STATE-10 · Treat intentional zero as an achievement
**DO:** Distinguish "you finished everything" from "there is nothing here", and present the former as
a success.
*Why:* Reaching zero is the goal in an inbox, a task list, or a review queue. Showing the same grey
placeholder as an unconfigured account throws away the best moment in the product.
*Applies:* queues, inboxes, task lists — anywhere zero is the target
*Check:* clear the last item. Does the screen acknowledge it?
*Severity:* recommended

### R-STATE-11 · Every section owns its own data
**DO:** Give each independently sourced region its own fetch, its own loading state, its own error
state, and its own retry.
*Why:* Regions on one screen come from different services at different speeds. Coupling them means
the slowest one sets the pace and the least reliable one sets the ceiling.
*Applies:* any screen assembling data from more than one source
*Check:* make one request hang. Does anything else on the page still work?
*Severity:* required

### R-STATE-12 · One failure must not blank the page
**DON'T:** Replace an entire screen with an error because a single section failed.
*Why:* The rest of the data arrived and is still useful. Discarding it turns a partial outage into a
total one.
*Applies:* screens with more than one data region
*Check:* force one request to reject. Is the remaining content still readable and usable?
*Severity:* required

### R-STATE-13 · Show what you already have while refreshing
**DO:** Render cached or previously loaded content immediately, then replace it when fresh data lands.
*Why:* The wait disappears from the user's experience entirely, and stale-but-visible beats
correct-but-blank for content that tolerates a few minutes of age.
*Applies:* feeds, lists, dashboards — not balances, prices, or anything where stale is misleading
*Check:* revisit a screen you have already loaded. Does it show content before the network returns?
*Severity:* recommended

### R-STATE-14 · Sequence the first run
**DO:** On a brand-new account, present setup as ordered steps with visible progress rather than a
single call to action.
*Why:* The first empty screen is where a new user decides whether the product is worth the effort.
One button answers "what now" but not "how much is left"; a short visible sequence shows the end from
the beginning and gives each completed step a reason to continue.
*Applies:* first-run dashboards, workspace creation, onboarding — not routine empty states later
*Check:* create a fresh account. Can you tell how many steps stand between you and a working setup?
*Severity:* recommended

### R-STATE-15 · A control has six states, not one
**DO:** Decide what a button looks like when idle, hovered, focused, held down, working, and
unavailable — then build the ones that apply to the platform you ship on.
*Why:* A control that looks identical before, during, and after a press tells the user nothing, so
they press it again. The duplicate submit that follows is not user error; it is the missing state.
*Applies:* buttons, icon buttons, and any control that triggers an action
*Check:* tab to it — visible focus ring? Hold it down — anything change? Trigger its slow path — does
the control itself say it is working? Then tap it on a phone and scroll away: a hover style that
latches after the finger lifts reads as a bug, because touch has no cursor to leave.
*Severity:* required
