# casticr-generator

Generates a `.casticr` file for a set of demo applications every day using
CAST Imaging, and stores each one as a GitHub Actions artifact.

## How it works

[`.github/workflows/casticr-scan.yml`](.github/workflows/casticr-scan.yml) runs
**daily at 2 PM IST (08:30 UTC)** and can also be started manually. For each
project in the matrix, in its own parallel job, it:

1. clones the repo (shallow),
2. runs `castimaging/imaging-docker-analyzer:latest -ak <key> -n <app>`,
3. finds the generated `<app>.casticr`
   (`results/<app>/<timestamp>/deep-analysis/<app>.casticr`),
4. uploads it as an artifact named `<app>.casticr`.

`fail-fast: false`, so one project failing does not cancel the others.

## Projects scanned

| App name           | Repository |
| ------------------ | ---------- |
| `AppPython`    | https://github.com/public-apis/public-apis |
| `AppMainframe` | https://github.com/aws-samples/aws-mainframe-modernization-carddemo |
| `AppDOTNET`    | https://github.com/razorpay/razorpay-dotnet-testapp |
| `AppJEE`       | https://github.com/spring-petclinic/spring-petclinic-microservices |

To add or remove a project, edit the `matrix.include` list in the workflow —
each entry is a `url` (public clone URL) and a `name` (the `-n` app name, also
the artifact name).

## Required secret

Add one Actions secret — the analysis key passed to `-ak`:

```bash
gh secret set IMAGING_ANALYSIS_KEY --repo <owner>/casticr-generator
```

## Getting the results

Open the latest **Daily .casticr scan** run under the **Actions** tab and
download the `<app>.casticr` artifacts. They use a stable name and short
retention, so each day's run refreshes them.
