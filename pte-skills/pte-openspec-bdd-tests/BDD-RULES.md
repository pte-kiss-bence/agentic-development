# BDD / Gherkin rules

The checklist for every Scenario this skill writes. Grounded in the official Cucumber guidance — [Writing better Gherkin](https://cucumber.io/docs/bdd/better-gherkin), [Anti-patterns](https://cucumber.io/docs/guides/anti-patterns), [Gherkin reference](https://cucumber.io/docs/gherkin/reference).

## The rules

1. **Declarative, not imperative.** Describe _what_ behaviour is expected, not _how_ the user drives the UI. Declarative steps survive UI and logic changes; procedural ones break on every redesign.
   - Yes: `When Free Frieda logs in with her valid credentials`
   - No: `When I type "frieda@x.com" in the email field / And I press "Submit"`

2. **One Scenario, one behaviour.** A Scenario illustrates a single rule. If it needs two unrelated outcomes, split it.

3. **Given / When / Then have fixed roles.** `Given` = context and prior state. `When` = the single event under test. `Then` = an observable outcome. Don't assert in `Given`; don't act in `Then`.

4. **One `When` per Scenario.** Multiple `When`s mean you're testing more than one behaviour, or you've slipped into imperative steps. Extra real outcomes go under `Then … And …`.

5. **No conjunction steps.** A single step doing two things (`Given I have shades and a brand new Mustang`) hides a seam — split into two steps, or use `And`.

6. **Third-person, business language.** Use the project's ubiquitous language and concrete personas, not `I`. Specs in another language stay in that language (set `# language: <code>` on line 1).

7. **`Scenario Outline` for varying data.** Same behaviour over many inputs → one `Scenario Outline` with an `Examples:` table, not copy-pasted Scenarios.

8. **`Background` for shared `Given`s.** A precondition common to every Scenario in a file goes in `Background:` once, not repeated.

9. **`Rule:` groups a business rule.** Use it to hold the Scenarios that illustrate one Requirement — the natural home for an OpenSpec `### Requirement`.

10. **`Feature` gets a description.** A short line of intent under the `Feature:` name ("In order to … As a … I want …", or free text) — it is the capability's living documentation.

11. **Scenarios are independent.** No Scenario may depend on another having run first; each sets up its own `Given`.

## Anti-patterns — reject these

- Procedural / UI steps (`visit "/login"`, `click the button`) — implementation, not behaviour.
- Conjunction steps (`… and …` inside one step).
- Long Scenarios padded with incidental detail unrelated to the behaviour.
- Scenarios named `test …` or `check …` — name the behaviour instead.
- Multiple `When … Then … When … Then` chains in one Scenario.
- Asserting outcomes in `Given`, or performing actions in `Then`.
