# casticr-generator

Generates a `.casticr` file for a set of demo applications using CAST Imaging,
and stores each one as a GitHub Actions artifact.

## How it works

Two workflows:

- [`casticr-scan.yml`](.github/workflows/casticr-scan.yml) holds the triggers,
  the project list and a `plan` job that decides what to run.
- [`casticr-analyze.yml`](.github/workflows/casticr-analyze.yml) is a reusable
  workflow that analyzes one project: one job per analyzer, each of which
  clones the repo, runs the analyzer, finds the `.casticr` and uploads it.

In the run graph each project gets its own box, with its `offline` and/or
`online` job inside. `fail-fast: false` everywhere, so one failure does not
cancel the rest.

### Analyzers

| Analyzer  | Image | Needs |
| --------- | ----- | ----- |
| `offline` | `castimaging/imaging-docker-offline-analyzer:latest -n <app>` | nothing |
| `online`  | `castimaging/imaging-docker-analyzer:latest -ak <key> -n <app>` | `IMAGING_ANALYSIS_KEY` secret, `dev.castimaging.io` |

### When it runs

| Trigger | Analyzer | Projects |
| ------- | -------- | -------- |
| Schedule: 8 PM IST (14:30 UTC), Mon–Fri | `offline` | all |
| Manual: Actions → **Scheduled .casticr scan** → Run workflow | `online` (default), `offline` or `both` | the name typed in `project`, or all if blank |

A `project` name that isn't in the list fails the `plan` job immediately and
prints the valid names.

## Projects scanned

Edit the `PROJECTS` list at the top of
[`casticr-scan.yml`](.github/workflows/casticr-scan.yml). Each entry is a
`name` (the `-n` app name, the artifact name, and what you type in `project`)
and a `url` (public clone URL). Nothing else needs changing.

## Required secret

Only the online analyzer needs it:

```bash
gh secret set IMAGING_ANALYSIS_KEY --repo <owner>/casticr-generator
```

## Getting the results

Open the run under the **Actions** tab and download the artifacts:

- offline: `<app>.casticr`
- online: `<app>-online.casticr`

They use stable names and 3-day retention, so each run refreshes them.
