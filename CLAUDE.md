# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

A React static website deployed to AWS S3 + CloudFront via a CloudFormation-managed CI/CD pipeline (CodePipeline + CodeBuild). There are two separate CloudFormation stacks: one for the site infrastructure and one for the pipeline.

## Commands

```bash
npm start        # Local dev server
npm run build    # Production build → build/
npm test         # Run tests (jsdom environment via react-scripts)
```

## Architecture

### Two CloudFormation Stacks

**`cfn/static-site.yml`** — the hosting infrastructure:
- S3 bucket for assets (public read via bucket policy)
- CloudFront distribution (HTTPS, HTTP/2, compression, Price Class 100)
- Origin Access Identity to lock direct S3 access
- Route53 alias record pointing the subdomain to CloudFront

Parameters: `RootDomainName`, `SiteDomainName`, `CertificateARN` (ACM cert must be in us-east-1)

**`cfn/pipeline.yml`** — the CI/CD pipeline:
- CodePipeline with three stages: Source (GitHub) → Build → Deploy
- Two CodeBuild projects: one for building (`buildspec.yml`), one for deploying (`deployspec.yml`)
- IAM roles scoped to S3 artifact bucket + site bucket + CloudFront invalidations
- S3 artifact bucket for pipeline state

Parameters: `SiteDomainName`, `CloudFrontDistribution`, `Pipeline`, `GitHubOwner`, `GitHubRepo`, `GitHubBranch`, `GitHubToken`

### Build and Deploy Specs

**`buildspec.yml`**: installs dependencies, runs tests, then `npm run build` — outputs the `build/` directory as the artifact.

**`deployspec.yml`**: three-step deploy to avoid race conditions between old `index.html` referencing deleted hashed assets:
1. Sync everything *except* `index.html` to S3 with `Cache-Control: max-age=31536000` (1 year)
2. Copy `index.html` separately with `Cache-Control: max-age=60, s-maxage=31536000` (1 min browser / 1 year CDN)
3. Invalidate CloudFront cache for `/index.html` only

This caching strategy works because react-scripts embeds content hashes in CSS/JS filenames — only `index.html` needs frequent invalidation.
