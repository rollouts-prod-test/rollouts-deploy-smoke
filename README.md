# rollouts-deploy-smoke

A tiny repository that produces **real GitHub Deployments** so Cursor Rollouts
(the Factory change monitor) can be tested end to end against GitHub
`deployment` / `deployment_status` webhooks.

## What happens on every push to `main`

`.github/workflows/record-production-deployment.yml` runs and:

1. creates a GitHub Deployment for the pushed commit in environment
   `production` (`production_environment: true`),
2. posts an `in_progress` deployment status,
3. posts a `success` deployment status a few seconds later.

Nothing is deployed anywhere. The repository exists only to emit the webhooks.
Each step above is a `deployment_status` webhook delivery to any GitHub App
installed on this org with the *Deployments* permission, including the Cursor
GitHub App, whose `deployment_status` handler records a Rollouts deployment
when the owning Cursor team has Rollouts enabled and the
`rollouts_github_deployment_ingest` gate on.

## Producing another deployment

Any commit to `main` works. The usual way is to bump `deploys/VERSION`:

```sh
printf '%s\n' "$(( $(cat deploys/VERSION) + 1 ))" > deploys/VERSION
git commit -am "deploy: bump version"
git push
```

Or run the workflow by hand: **Actions → Record production deployment → Run
workflow** (it records a deployment for the current `main` head).

Deployment history: `https://github.com/<owner>/rollouts-deploy-smoke/deployments/production`.

## Moving this repository

The workflow only needs `permissions: deployments: write` on the repository's
own `GITHUB_TOKEN`, so it works unchanged in any GitHub org or account that has
Actions enabled. Cursor binds a GitHub App installation to a single Cursor
team, and only *organization* installations bind (personal-account installs
are skipped), so for a fresh Cursor test team the repo must live in a GitHub
org that is not already connected to another Cursor team.
