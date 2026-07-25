# Phase 1 — Frontend on S3 + CloudFront

Migrated the Hypothesis Log frontend off Vercel and onto AWS, serving a private S3 bucket through CloudFront over HTTPS. Done as part of a fixed 21-day Free-plan sprint, so the emphasis was on getting real hands-on experience with the core services and capturing durable evidence (this write-up + screenshots + committed configs), since the live URL disappears when the account auto-closes on Aug 10, 2026.

## What was built

- Private S3 bucket (`hypothesis-log-frontend-awslearning`, ca-central-1) holding the Vite production build. No static website hosting, no public access.
- CloudFront distribution (`d2remctp8dpvdy.cloudfront.net`) in front of it, using Origin Access Control (OAC) so the bucket stays private and only this distribution can read it.
- SPA routing handled via custom error responses (403/404 -> `/index.html` with a 200) so a browser refresh on a client-side route doesn't 404.
- Frontend pointed at the existing Render backend through a build-time env var; backend CORS updated to allow the new CloudFront origin.

## Architecture

```
User browser
    |
    v  (HTTPS)
CloudFront distribution  d2remctp8dpvdy.cloudfront.net
    |  (OAC — signed requests, bucket stays private)
    v
S3 bucket (private)  hypothesis-log-frontend-awslearning
    - index.html, hashed JS/CSS assets, favicon

API calls go browser -> Render backend directly (NOT through CloudFront):
Browser --(CORS)--> https://hypothesis-log.onrender.com
```

## Services and concepts used

- **AWS:** S3, CloudFront, IAM, Origin Access Control
- **Security:** private bucket + OAC, auto-generated bucket policy scoped to the distribution ARN, IAM user (`boris-dev`) instead of root for daily work, MFA on both root and the IAM user
- **Frontend:** Vite production build, build-time env var embedding, SPA error-page fallback
- **Networking:** CORS preflight (OPTIONS), `Access-Control-Allow-Origin`, preflight caching (`access-control-max-age`), CloudFront cache invalidation
- **Tooling:** AWS CLI (`s3 mb`, `s3 sync`, `cloudfront create-invalidation`, `sts get-caller-identity`), curl for header inspection, Chrome DevTools Network tab

## Design decisions worth defending in an interview

**Why the bucket is private (OAC) instead of a public static-website bucket.**
The common tutorial approach is to enable S3 static website hosting and make the bucket public. Instead, the bucket is fully private and CloudFront reaches it via OAC, which signs requests so only this distribution can read the objects. This is AWS's current recommended pattern and keeps the origin off the public internet entirely.

**Why WAF was declined.**
The CloudFront wizard defaults to enabling WAF, which is billed (~$14 per 10M requests). For a learning deployment with no real traffic it adds cost without value, so it was turned off.

## Debugging postmortem: a CORS failure that wasn't a CORS config problem

The frontend loaded fine but every API call failed with "No 'Access-Control-Allow-Origin' header". The backend CORS config was correct and a `curl` OPTIONS preflight from the terminal returned all the right headers — yet the browser kept failing, including in incognito and in a second browser.

Root cause: a one-character typo in the CloudFront domain (`...dpdvy` vs `...dpvdy`). The wrong string had been copied into the backend's `ALLOWED_ORIGINS` **and** into the `curl` test's `Origin` header, so the two agreed with each other while disagreeing with the browser's real origin.

Lessons that generalize:
- A passing `curl` preflight only proves the backend allows whatever origin `curl` *claimed* to be. It says nothing about the browser's actual origin unless the two strings are identical.
- Don't retype resource identifiers. Pull them from the source of truth:
  `aws cloudfront list-distributions --query "DistributionList.Items[].DomainName" --output text`
- In DevTools, "Response headers (0)" plus "Provisional headers are shown" means the request never completed — distinct from a completed response that's missing the CORS header.
- Preflight results are cached (`access-control-max-age`), so a fixed backend can still look broken for minutes; test in incognito or with "Disable cache".

## Also learned (noted, not yet acted on)

- **IAM user vs IAM Identity Center.** Long-lived access keys on an IAM user are convenient but are an anti-pattern for human users at scale; Identity Center issues short-lived credentials instead. Fine for a solo learning account, but worth knowing the distinction.
- **FullAccess policies are not least privilege.** The `boris-dev` user has AWS-managed `*FullAccess` policies attached for convenience. A real environment would scope these down to only the actions actually used.

## Cost

Effectively zero. A few hundred KB in S3 and a handful of CloudFront requests, both far under the Always Free allowances. No WAF, no NAT Gateway, no compute.
