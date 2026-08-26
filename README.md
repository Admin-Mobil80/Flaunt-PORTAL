# Flaunt Portal

Customer-facing static frontend for Flaunt. Plain HTML/CSS/JS in [`public/`](public/), no build step. Deploys to the shared S3 bucket `webapps.flaunt.network` (under the `/PORTAL` prefix) behind its own CloudFront distribution at `flaunt.network`.

This repo's CloudFormation stack (`infrastructure/template.yaml`) **owns the shared bucket** — it creates it, and [Flaunt-BMS](https://github.com/Admin-Mobil80/Flaunt-BMS) imports it by reference. **This stack must be deployed at least once before Flaunt-BMS's first deploy.**

## One-time account setup (do this before the first push to `main`)

1. Fix/confirm your local AWS CLI credentials (`aws sts get-caller-identity`).
2. From [Flaunt-BACKEND](https://github.com/Admin-Mobil80/Flaunt-BACKEND), manually deploy `bootstrap/github-oidc.yaml` once:
   ```bash
   aws cloudformation deploy \
     --template-file bootstrap/github-oidc.yaml \
     --stack-name flaunt-github-oidc \
     --capabilities CAPABILITY_NAMED_IAM
   ```
   This creates the GitHub OIDC provider and all three repos' deploy roles.
3. Look up your Route 53 hosted zone ID for `flaunt.network`:
   ```bash
   aws route53 list-hosted-zones-by-name --dns-name flaunt.network
   ```
4. In [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml), replace the two placeholders:
   - `DEPLOY_ROLE_ARN` — the `flaunt-portal-deploy-role` ARN from step 2's stack output
   - `HOSTED_ZONE_ID` — the value from step 3
5. Push to `main`. CI will create the bucket, cert, distribution, and DNS record.

Deploy order across the three repos: **Portal → BMS** (BMS depends on Portal's exports) — **Backend** is independent and can deploy any time after step 2.

## Local development

Just open `public/index.html` in a browser, or serve the folder with any static file server.
