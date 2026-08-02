# States

Loading, empty, and partial. Error content lives in `feedback.md`; this file covers whether a state
exists at all and which form it takes.

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

### R-STATE-03 · No loader for waits under a second
**DON'T:** Show a spinner for an operation that typically completes in under ~1 second.
*Why:* It appears and vanishes before the animation reads as motion, so it registers as a glitch and
makes the interaction feel *less* smooth than showing nothing.
*Applies:* fast local operations, cached reads, optimistic updates
*Check:* time the operation. Under a second — render the result directly.
*Severity:* recommended

### R-STATE-04 · Give long waits something to say
**DO:** Add text to a wait that runs past a few seconds, and change that text as the work progresses.
*Why:* An indefinite animation with no words starts reading as stuck. Text that advances shows the
system is still working, and people wait substantially longer for it.
*Applies:* multi-step or network-bound operations
*Check:* is there any point past ~5 seconds where the screen says nothing about what is happening?
*Severity:* recommended

### R-STATE-05 · Past ten seconds, stop looping
**DO:** Switch to determinate progress or an explicit step list for waits that can exceed ~10 seconds.
*Why:* A loop that never resolves stops reassuring and starts irritating — the user cannot tell
progress from a hang, and patience inverts.
*Applies:* uploads, imports, builds, batch jobs
*Check:* can this operation exceed 10 seconds? Then it needs progress, not a spinner.
*Severity:* recommended

### R-STATE-06 · Match the loader to the shape of the wait
**DO:** Use a skeleton that mirrors the layout when a whole region is arriving; use an inline spinner
inside a button or small control when a single action is in flight.
*Why:* A skeleton lets the eye settle into the layout before content lands. A skeleton inside a
button is noise; a page-wide spinner throws away the layout information you already have.
*Applies:* any loading affordance
*Check:* does the loader occupy the same footprint the real content will?
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
