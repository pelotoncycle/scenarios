# scenarios

Scenario registry for the Peloton scenario-driven quality system. Contains the YAML scenario definitions that are evaluated against live environments.

Scenarios are the **holdout set** — they define expected product behavior from a user perspective and live outside the application codebase so neither developers nor AI coding agents can modify them to game CI.

## Structure

```
SCN-WWW-001.yaml   — www app scenarios
SCN-APP-001.yaml   — native app scenarios
...
```

## Running scenarios

Use [pelotoncycle/scenario-runner](https://github.com/pelotoncycle/scenario-runner) to execute scenarios against a live URL.

## Writing scenarios

See the runner repo for the full format guide and tips.

## Access

This repo is branch-protected. Application repos have read access only — no write access from code repos or AI coding agents.
