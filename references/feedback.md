# Feedback

What the interface says when something fails, and what it says when something works. Whether a state
exists at all is covered in `states.md`.

---

### R-FEEDBACK-01 · Never put internal detail on screen
**DON'T:** Render a raw exception, stack trace, database message, or backend identifier to an end user.
*Why:* It is unreadable to the person who has to act on it, and it discloses your internal structure
to everyone else. This is a security control as much as a usability one.
*Applies:* every user-facing error path
*Check:* force a backend failure. Does any part of the message come from the framework rather than
from you?
*Severity:* required

### R-FEEDBACK-02 · No silent failures
**DO:** Make every failed action produce a visible response.
*Why:* A button that does nothing is the worst possible outcome — the user cannot tell whether it
worked, whether it broke, or whether they should try again, so they retry and risk duplicating it.
*Applies:* every action that can fail
*Check:* force each action to fail with the console closed. Did the screen change?
*Severity:* required

### R-FEEDBACK-03 · An error names what, why, and what next
**DO:** Say what failed, give the reason in the user's terms, and offer the action that resolves it.
*Why:* Generic apologies leave the user unable to act and unsure whether the operation partially
completed. After a payment attempt, "something went wrong" does not answer the only question that
matters: was I charged?
*Applies:* every user-facing error message
*Check:* read the message aloud. Does it answer all three questions without the console?
*Severity:* required

### R-FEEDBACK-04 · Put the error where the problem is, and take the user there
**DO:** Place the message next to the field, control, or region that failed — and on a rejected
submit, move focus and scroll to the first failure rather than only marking it.
*Why:* The user's attention is already on the thing they touched. Distance between a problem and its
explanation is work handed to them, and on anything taller than one screen that work is a search.
Landing them on the field turns the search into a correction.
*Applies:* validation errors, action failures, section-level failures
*Check:* can the message be seen without scrolling from its cause? Blank a field near the top of a
long form and submit from the bottom — does the page take you there?
*Severity:* required

### R-FEEDBACK-05 · Toasts only for what can be missed
**DON'T:** Deliver anything the user must act on through a message that dismisses itself.
*Why:* A toast is unmissable only if you happen to be looking. If missing it leaves the user stuck or
misinformed, the channel is wrong.
*Applies:* transient notifications
*Check:* if the user looked away for five seconds, are they still fine? If not, use inline or a modal.
*Severity:* recommended

### R-FEEDBACK-06 · Blocking requires a way forward
**DO:** Reserve modals for failures the user genuinely cannot continue past, and always give the
action that unblocks them.
*Why:* Taking over the screen is the strongest interruption available. Spending it on something
dismissible trains people to dismiss; spending it without an exit creates a dead end.
*Applies:* payment failures, permission denials, expired sessions, destructive confirmations
*Check:* does the modal contain a control that resolves the situation, not just "OK"?
*Severity:* required

### R-FEEDBACK-07 · Confirm every completed action
**DO:** Make the result of a completed action visible.
*Why:* Without confirmation the user is left guessing whether it took effect, and the natural
response to that doubt is to do it again — which is how you get duplicate submissions.
*Applies:* submits, saves, payments, destructive actions, state changes
*Check:* complete the action and cover the console. Is it obvious that it worked?
*Severity:* required

### R-FEEDBACK-08 · Let the change itself be the confirmation
**DO:** Prefer a visible state change over an added message when the change is unambiguous.
*Why:* A card that moves to another column and stays there has already answered the question. A
banner on top of it is noise, and noise makes real confirmations easier to ignore.
*Applies:* direct manipulation, toggles, inline edits, reordering
*Check:* is the change visible on its own? Then no extra message is needed.
*Severity:* recommended

### R-FEEDBACK-09 · Match the celebration to the moment
**DON'T:** Give routine actions the treatment reserved for milestones.
*Why:* Emphasis is a budget. Spend it on saving a draft and there is nothing left for completing
onboarding or shipping a first project.
*Applies:* success animations, full-screen confirmations, celebratory copy
*Check:* would this feel excessive on the twentieth repetition? Then it is too much for the first.
*Severity:* recommended
