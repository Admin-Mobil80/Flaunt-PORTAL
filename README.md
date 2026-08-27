# Flaunt Portal

Static frontend for **https://flaunt.network**. One self-contained `public/index.html` — a
hash-routed single page, no build step, no dependencies.

## How this deploys

**Push to `main`.** [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml) assumes
`flaunt-github-deploy-prod` through GitHub OIDC (no stored AWS keys), syncs `public/` to S3,
invalidates CloudFront, waits for the invalidation, then checks the site returns 200.

The division of labour is fixed:

| Owns | What |
|---|---|
| **This repo, via GitHub Actions** | the site's content |
| **[Flaunt-BACKEND](../Flaunt-BACKEND), via CDK** | the bucket, distribution, certificate, DNS, and the IAM role this workflow assumes |

CDK never uploads content, and this workflow never creates infrastructure. A copy change
ships by pushing here; it needs no CloudFormation deploy and no AWS credentials on your machine.

## Editing the page

`public/index.html` is **generated** — do not hand-edit it. Both frontends are built from one
source so they cannot drift apart:

```bash
cd ../design && AWS_PROFILE=cloudmeter node build-site.mjs
```

That writes this repo's `public/index.html` and Flaunt-BMS's. Commit the result and push.

## Current state

This is a **preview build of a design prototype**. The page carries a banner saying so and is
served with `X-Robots-Tag: noindex`, because there is no backend behind it yet — no accounts,
no payments, nothing persisted. It exists so the product can be reviewed and clicked through
while the real services are built.
