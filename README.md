# casticr-generator

Generates `.casticr` files for a set of applications using CAST Imaging and
stores each one as a GitHub Actions artifact.

## Files you edit

| File | What it controls |
| ---- | ---------------- |
| [`repos.txt`](repos.txt) | Which repositories to scan: one `<clone URL> [app name]` per line. The name is optional and is taken from the URL if left out. |
| [`casticr-config.yml`](casticr-config.yml) | What the scheduled run does (analyzer, image tag, repos), the per-analyzer time limit, the online cloud URL, and artifact retention. |
| [`casticr-scan.yml`](.github/workflows/casticr-scan.yml) | Only the schedule time (cron line). |
| [`extend-speed.yml`](.github/workflows/extend-speed.yml) | Manual test: times one extension download from CAST Extend on a runner (needs the `EXTEND_API_KEY` secret). |

## Analyzers

Both use the same image tag (`dev` by default):

| Analyzer  | Image | Needs |
| --------- | ----- | ----- |
| `offline` | `castimaging/imaging-docker-offline-analyzer:<tag>` | nothing |
| `online`  | `castimaging/imaging-docker-analyzer:<tag>` | `IMAGING_ANALYSIS_KEY` secret |

## Running

- **Scheduled:** 8 PM IST (14:30 UTC), Mon–Fri, using `casticr-config.yml` → `schedule`
  (today: offline, tag `dev`, all repos).
- **Manual:** Actions → **Scheduled .casticr scan** → Run workflow:
  - `analyzer`: `online` (default), `offline` or `both`
  - `repos`: comma-separated app names or URLs; blank = all. A URL that isn't
    in `repos.txt` is scanned ad hoc.
  - `tag`: `dev` (default), `int` or `latest`

## How a run works

1. **plan** reads `repos.txt`, `casticr-config.yml` and the form, and builds
   the job list. Unknown repo names fail here, with the valid names listed.
2. **scan**: one job per repo × analyzer, all in parallel, named
   `<app> / offline` and `<app> / online`. Each clones the repo, runs the
   analyzer and uploads the result. One failure doesn't stop the others.
3. **merge extensions cache** (only when online ran). All online scans read
   one shared extensions cache. Each hands back only the extensions it newly
   downloaded, and this job merges them and saves the next version of the
   shared cache (older versions are deleted). The first online run starts with an empty cache.

## Results

Download from the run's **Artifacts**:

- offline: `<app>.casticr`
- online: `<app>-online.casticr`

## Required secret

Only the online analyzer needs it:

```bash
gh secret set IMAGING_ANALYSIS_KEY --repo <owner>/casticr-generator
```
