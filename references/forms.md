# Forms

Nobody enjoys filling in a form. Every rule here removes a reason to abandon one. Error *wording* and
placement are in `feedback.md`.

---

### R-FORM-01 · Do not disable the submit to enforce validity
**DON'T:** Grey out a submit control to communicate that the form is not finished.
*Why:* It hands the user the job of finding what is missing, on a form mixing required and optional
fields — and disabled controls are skipped by keyboard and screen-reader navigation, so the button
and the reason it is off are both hard to discover. Disabling is right only where the reason is
obvious (`Previous` on page one) or the work is in flight and a second press would submit twice.
*Applies:* any form gating submission on validity
*Check:* would the user have to work out how to enable it? Then leave it enabled and let the click
name what is missing.
*Severity:* required

### R-FORM-02 · Mark what is required
**DO:** Indicate required fields before the user reaches the end of the form.
*Why:* Otherwise required-ness is discovered by failing, which means the user learns the rules by
breaking them.
*Applies:* any form mixing required and optional fields
*Check:* can a first-time user tell which fields are mandatory without submitting?
*Severity:* required

### R-FORM-03 · Validate when the field is left, not at submit
**DO:** Check a field as soon as the user moves away from it.
*Why:* Holding every complaint until submit means filling everything in, waiting, then hunting back
up the page for the one thing that was wrong — and the further down the form, the more work is at
risk.
*Applies:* fields with a format or uniqueness constraint
*Check:* type an invalid email and click elsewhere. Do you find out now, or after submitting?
*Severity:* required

### R-FORM-04 · Show limits while typing
**DO:** Display remaining characters on any length-limited field as the user types.
*Why:* Discovering a limit after writing a paragraph means deleting half of it — work you invited and
then destroyed.
*Applies:* fields with a maximum length
*Check:* is the limit visible before it is reached?
*Severity:* recommended

### R-FORM-05 · Never ask for what you already know
**DO:** Prefill values already held — account email, saved address, current settings.
*Why:* Retyping known data is pure friction, and every retyped field is another chance to introduce a
typo that fails validation.
*Applies:* authenticated flows, repeat checkouts, settings screens
*Check:* is any prefillable field starting empty for a signed-in user?
*Severity:* recommended

### R-FORM-06 · Show credential rules being satisfied
**DO:** List password or similar requirements up front and mark each as it is met.
*Why:* Composing a password against invisible rules is guesswork, and finding out at submit means
starting over on the one field people already resent.
*Applies:* password creation, any field with composition rules
*Check:* start typing. Do the requirements respond?
*Severity:* recommended

### R-FORM-07 · Accept the input, fix the format yourself
**DO:** Take phone numbers, card numbers, dates and postcodes in whatever shape they arrive, and
normalise them server-side.
*Why:* Rejecting a correct value because of punctuation is the machine asking the human to do the
machine's job.
*Applies:* any field with a conventional but variable written format
*Check:* enter the same value three ways. Are all three accepted?
*Severity:* recommended

### R-FORM-08 · Break long forms into steps
**DO:** Split a form past roughly seven fields into shorter sequential steps with visible progress.
*Why:* A long form is judged before it is started. A short first step gets people moving, and the
commitment already made carries them through the rest.
*Applies:* signup, checkout, onboarding, applications
*Check:* count the fields on screen at once. Past ~7, is there a reason they are not staged?
*Severity:* recommended
