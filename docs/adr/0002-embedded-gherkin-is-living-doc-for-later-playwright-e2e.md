# Embedded Gherkin is living documentation, implemented later as Playwright E2E

The Gherkin `pte-openspec-bdd-tests` embeds in each story card's `BDD Test` section is **living documentation now** — a human-readable, declarative description of the story's behaviour (specification by example) for shared understanding and refinement. It is **not executable inside this pipeline**. In a **later, separate phase** it is implemented as a **Playwright E2E** test; that phase has no skill yet and is marked "planned" in the pipeline docs.

Consequences, each a deliberate scope boundary for `pte-openspec-bdd-tests`:

- **It writes zero new files.** It only fills the `BDD Test` placeholder in existing story cards, in place. There is no separate `.feature` tree, and there are **no Cucumber step-definition stubs** — the eventual runner is Playwright E2E, not Cucumber glue, so generating step-defs here would be scaffolding for the wrong target.
- **One mode only.** If no story cards exist, the skill stops and offers to run `pte-openspec-to-stories` first. It does **not** fall back to standalone `.feature` output — that would break the "0 new files" invariant.
- **The declarative rule (`BDD-RULES.md`) is unaffected.** The embedded Gherkin stays UI-free; the Playwright/UI detail lives in the future E2E glue layer, never in the Gherkin.

This is why the skill's own prose calls the output "living documentation" and not "executable" — the "executable" step is deferred to the Playwright phase, so earlier wording that called it "executable living documentation" was corrected.
