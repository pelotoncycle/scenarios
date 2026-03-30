# scenarios — Agent Instructions

This is the scenario registry for the Peloton scenario-driven quality system. It contains only YAML scenario definitions — no application code, no runner logic.

**This repo is the holdout set.** Scenarios define expected product behavior from a user perspective and must remain independent of any application codebase. Do not modify scenarios to make CI pass — fix the product instead.

## Adding a scenario

Scenarios live at the root of the repo, named `SCN-<APP>-<NNN>.yaml` (e.g. `SCN-WWW-001.yaml`).

### Format

```yaml
scenario:
  id: SCN-XXX-001
  title: "Human-readable title"
  app: www
  criticality: P0   # P0 | P1 | P2  — sets the pass threshold (0.95 / 0.80 / 0)
  steps:
    - dismiss any cookie or country dialog if present
    - hover over "Bikes" in the top navigation bar
    - click the "Shop Bike" button in the dropdown menu that appears
    - scroll down to see the product details
    - fill: { selector: '[data-test-id="shippingPhone"]', value: '5555555555' }
    - fill: { frame: 'iframe[name*="privateStripeFrame"][name*="card-number"]', selector: '[name="cardnumber"]', value: '4242424242424242' }
  acceptance_criteria:
    - a product name is visible on the page
    - a price in USD is displayed
    - an Add to Cart button is visible
    - the page is not visually showing an error state or 404 page
```

### Step types

| Syntax | Behaviour |
|---|---|
| Plain string | AI (Stagehand) interprets the instruction and acts on the live page |
| `fill: { selector, value }` | Direct Playwright fill — no AI, for fields the AI can't reliably access |
| `fill: { frame, selector, value }` | Direct fill inside an iframe (e.g. Stripe payment fields) |
| `wait: 5000` | Explicit pause in milliseconds — use when async content needs time to render |
| `goto: /path` | Direct navigation — avoid unless content testing only |

### Writing reliable steps

- **Use natural language for all navigation.** Scenarios test real user journeys — if a nav link is broken, the test should catch it.
- **Be specific about elements and context.** Bad: _"go to the bike page"_. Good: _"click the 'Shop Bike' button in the dropdown that appears under the Bike nav item"_.
- **One action per step.** Split compound interactions. Bad: _"click Bike in the nav"_ (AI must infer hover is needed first). Good: _"hover over 'Bike' in the top navigation bar"_ then _"click the 'Shop Bike' button in the dropdown"_. Splitting saves ~20s per compound step.
- Add a `dismiss … dialog` step first — test environments often show country/cookie modals.

### Writing reliable acceptance criteria

- Write criteria as **explicitly visual facts** — the judge evaluates against screenshots.
- Bad: _"the page is not showing an error state"_ — the judge may route console errors into this.
- Good: _"the page is not visually showing an error state or 404 page"_.
- Add a dedicated criterion for JS errors if you want to catch them explicitly.

### What qualifies as a scenario

A scenario earns its place only when **all three** are true:

1. A real user would notice if it broke
2. It requires the full stack to validate
3. "Correct" requires judgment — can't be reduced to a single boolean assertion

**Target counts per app:** P0: 5–10 max · P1: 15–25 · P2: post-merge/production monitoring only

### Passing thresholds

| Criticality | Required score |
|---|---|
| P0 | ≥ 0.95 |
| P1 | ≥ 0.80 |
| P2 | any |

## Running scenarios

Use [pelotoncycle/scenario-runner](https://github.com/pelotoncycle/scenario-runner).

## Fixing a failing scenario

1. Read the `reasoning` field in the result JSON — the judge explains what was missing.
2. If navigation failed: make the step more specific (add element name, context, or split into hover + click).
3. If a criterion is failing spuriously: tighten the wording to be more explicitly visual.
4. Re-run and verify the satisfaction score meets the criticality threshold.
