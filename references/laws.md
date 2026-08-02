# Laws

Four established principles, stated as things to do rather than things to know. Each has a named
origin, but the citation is not the point — the applied rule is.

---

### R-LAW-01 · Put conventional things where they are expected — Jacob's Law
**DO:** Place recurring components in the position users have already learned from every other product.
*Why:* People spend almost all their time in other applications, and arrive expecting yours to behave
like those. Novelty in a navigation element is not innovation; it is a tax on every visit.
*Applies:* carts, search, account menus, primary navigation, save and close controls
*Check:* name three well-known products in the same category. Is this component where theirs is?
*Severity:* recommended

### R-LAW-02 · Conventions are per-platform, not universal
**DO:** Re-derive the expected position for each platform you ship on, rather than porting one layout.
*Why:* The same product has different conventions on desktop and mobile, largely because the hand
holding a phone can reach the bottom of the screen far more comfortably than the top corner.
*Applies:* any product shipping on more than one form factor
*Check:* on a phone, one-handed, can the primary action be reached with a thumb?
*Severity:* recommended

### R-LAW-03 · Mirror the layout for right-to-left languages
**DO:** Flip directional layout, navigation and iconography for RTL locales rather than translating
text in place.
*Why:* In an RTL locale the expected position of a control is mirrored too. Translated text in an
LTR layout is a half-localised product that feels wrong without the user being able to say why.
*Applies:* any product shipping in Arabic, Hebrew, Persian or Urdu
*Check:* switch the locale. Did the layout mirror, or only the words?
*Severity:* recommended

### R-LAW-04 · Fewer competing choices — Hick's Law
**DO:** Reduce the options presented at one moment; let filtering and search reveal the rest.
*Why:* Decision time rises with the number and complexity of choices, and a screen of equally
weighted options is a screen with no recommendation. Curated entry points outperform exhaustive ones.
*Applies:* navigation, menus, pricing, category listings, dashboards
*Check:* count the things competing for the first click. Is one of them clearly primary?
*Severity:* recommended

### R-LAW-05 · One primary action per screen
**DO:** Give each screen a single visually dominant action, with everything else subordinate.
*Why:* Two equally emphasised actions is a question the user has to answer before doing anything, on
a screen where you already know which one they usually want.
*Applies:* any screen with more than one call to action
*Check:* squint at it. Does exactly one control stand out?
*Severity:* recommended

### R-LAW-06 · Reveal complexity as it is needed — progressive disclosure
**DO:** Show what is relevant now and keep the rest reachable but out of the way.
*Why:* Capability presented all at once reads as clutter and hides the thing the user actually came
for. Nothing is removed by deferring it; it simply stops competing.
*Applies:* settings, editors, creation flows, feature-dense tools
*Check:* is everything visible here needed to take the next step?
*Severity:* recommended

### R-LAW-07 · Do not bury a primary path
**DON'T:** Hide a main capability so deep that it needs a tutorial to find.
*Why:* Progressive disclosure fails in the other direction too. A feature nobody discovers was not
worth building, and the user cannot ask for something they do not know exists.
*Applies:* anything hidden behind menus, overflow controls, or keyboard-only entry points
*Check:* give someone the goal and watch. Do they find it without being told?
*Severity:* required

### R-LAW-08 · Absorb the complexity yourself — Tesler's Law
**DO:** Take irreducible complexity into the build rather than exporting it to the user.
*Why:* Complexity in a system is conserved; the only real question is who carries it. Time an
engineer spends once is time every user would otherwise spend repeatedly.
*Applies:* setup, configuration, data entry, anything with a sensible default
*Check:* for each thing you ask the user to decide — could the product work it out instead?
*Severity:* recommended
