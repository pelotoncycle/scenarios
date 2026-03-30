# scenarios

Scenario registry for the Peloton scenario-driven quality system. Contains the YAML scenario definitions that are evaluated against live environments.

Scenarios are the **holdout set** — they define expected product behavior from a user perspective and live outside the application codebase so neither developers nor AI coding agents can modify them to game CI.

## Structure

Scenarios are named `SCN-<APP>-<NNN>.yaml` (e.g. `SCN-WWW-001.yaml`), grouped by app.

## Running scenarios

Use [pelotoncycle/scenario-runner](https://github.com/pelotoncycle/scenario-runner) to execute scenarios against a live URL.

## Writing scenarios

### Format

```yaml
scenario:
  id: SCN-XXX-001
  title: "Human-readable title"
  app: www
  criticality: P0   # P0 | P1 | P2
  steps:
    - dismiss any cookie or country dialog if present
    - hover over "Bikes" in the top navigation bar
    - click the "Shop Bike" button in the dropdown menu that appears
    - scroll down to see the product details
  acceptance_criteria:
    - a product name is visible on the page
    - a price in USD is displayed
    - an Add to Cart button is visible
    - the page is not visually showing an error state or 404 page
```

### Tips

- **Use natural language for all navigation.** Scenarios test real user journeys — if a nav link is broken, the test should catch it.
- **Be specific about elements and context.** Bad: _"go to the bike page"_. Good: _"click the 'Shop Bike' button in the dropdown under the Bike nav item"_. The more precisely you describe the element, the faster and more reliably the AI executes it.
- **One action per step.** Split compound interactions — "hover over X" and "click Y in the dropdown" as separate steps is ~20s faster than combining them, because the AI doesn't have to infer that a hover is required first.
- **Write acceptance criteria as explicitly visual facts.** The judge evaluates against screenshots. Bad: _"the page is not showing an error state"_ — the judge may route console errors into this. Good: _"the page is not visually showing an error state or 404 page"_. Add a dedicated criterion for JS errors if you want to catch them explicitly.
- Add a `dismiss … dialog` step first — test environments often show country/cookie modals.

### What qualifies as a scenario

Scenarios are expensive — LLM inference cost, CI time, and flakiness risk all compound. A scenario earns its place only when **all three** of these are true:

1. A real user would notice if it broke (not an internal detail — a visible outcome)
2. It requires the full stack to validate (can't be adequately tested in isolation)
3. "Correct" requires judgment (acceptance criteria can't be reduced to a single boolean assertion)

**Use the right tool instead:**

| Thing to test | Right tool |
|---|---|
| Price calculation logic | Unit test |
| Component renders correctly | Component test / Storybook |
| API returns expected shape | Contract test |
| Page doesn't crash on load | Cypress smoke test |
| Visual layout matches design | Percy / Chromatic |
| Edge cases of a form or function | Unit + integration tests |

**Anti-patterns to avoid:**

- **Scenario creep** — adding a scenario every time a production bug is found. First ask: should this have been a unit test?
- **Happy-path-only** — a few well-chosen failure scenarios (payment timeout, out-of-stock mid-checkout) are worth more than five more happy paths
- **Implementation-coupled** — "user clicks the blue Confirm Order button" breaks every redesign; describe intent: "user completes checkout"
- **Overlapping coverage** — two scenarios exercising 90% of the same path should be merged

**Target counts per app:** P0: 5–10 max · P1: 15–25 · P2: post-merge/production monitoring only

### Passing thresholds

| Criticality | Required score |
|---|---|
| P0 | ≥ 0.95 |
| P1 | ≥ 0.80 |
| P2 | any |

### Why a satisfaction score instead of pass/fail

A score lets you set thresholds by criticality and surfaces degradation before it becomes a hard failure. Multiple runs produce a distribution rather than a single number — similar to how ML uses validation sets.

## Access

This repo is branch-protected. Application repos have read access only — no write access from code repos or AI coding agents.
