# PostHog NPM Package Compromise – November 24, 2025

On November 24, 2025, a compromised NPM publishing token was used to publish malicious versions of several PostHog JavaScript SDKs. The attacker leveraged the [Shai-Hulud](https://helixguard.ai/blog/malicious-sha1hulud-2025-11-24) supply-chain attack that affected ~300 publishers and over 25,000 public repositories.

The malicious versions were available for approximately 6 hours across nine distinct packages. While we have not identified evidence of the payload triggering for PostHog customers, we are continuing to investigate and have taken immediate remediation steps.

## Summary
One of our NPM publishing tokens was compromised and used to publish malicious versions of several PostHog JavaScript SDKs. These versions included lifecycle-triggered scripts consistent with the Shai-Hulud attack family.

While the scope was limited to our JavaScript packages, multiple product integrations depend on them, and malicious versions were briefly installable. We responded by unpublishing malicious versions, publishing safe replacements, rotating credentials, and initiating a broader hardening of our software supply chain.

We have not identified evidence of data-exfiltration or package-based event forwarding from compromised versions, but are advising customers to verify local environments and delete cached artifacts if affected.

## Timeline
**Nov 24 – 06:12 UTC** – First malicious package versions published  
**Nov 24 – 07:00 UTC** – Compromised token usage begins across dependent repos  
**Nov 24 – 09:36 UTC** – First internal alert: anomalous package contents reported  
**Nov 24 – 09:39 UTC** – Initial investigation confirms malicious code in multiple JS SDK packages  
**Nov 24 – 09:42 UTC** – Affected versions in main repo unpublished and patching initiated  
**Nov 24 – 10:28 UTC** – Broader compromise identified across additional package groups  
**Nov 24 – 13:10 UTC** – Full list of affected versions established  
**Nov 24 – 15:25 UTC** – Credential rotation and token revocation completed  
**Nov 24 – 16:32 UTC** – Public update published including list of safe replacement versions  
**Nov 25 – 02:40 UTC** – Completed remediation, confirmed no detected exfiltration  

## Root Cause Analysis
The compromise occurred due to three converging factors:

1. **Publishing token compromise**  
   A long-lived NPM token with broad package publishing permissions was accessed and misused by an external actor.

2. **Shai-Hulud supply chain exploit**  
   The attacker used a known mass-infection technique targeting active maintainers to propagate malicious scripts via npm install hooks.

3. **Insufficient pipeline isolation**  
   The JS SDK publishing pipeline did not enforce zero-trust boundaries, allowing compromised credentials to affect all dependent package release paths.

The impact window was extended by the scope of our package surface area on NPM and the absence of per-package cooldown checks before install availability.

## Impact
- 9 JavaScript SDKs and tools received malicious versions
- 0 confirmed customer executions of exfiltration behavior from compromised packages
- Potential risk to:
  - application environments installing affected versions
  - developer workstations running install scripts
  - CI and build pipelines using cached artifacts

SDKs in other languages (Python, Go, Rust, Java, etc.) were not affected.

## Remediation

### Immediate Actions (Completed)
- Revoked compromised NPM token
- Unpublished all malicious versions
- Published clean versions of impacted packages
- Required customers to upgrade to safe patch releases
- Completed credential rotation across maintainers

### Technical Controls That Will Prevent Future Incidents
- Move away from long-lived NPM tokens to **Trusted Publishers**
- Enforce 2FA for publishing on all packages org-wide (we already require 2FA on all npm accounts) 
- Mandate pnpm 10 with pre/post install scripts disabled
- 3-day cooldown before packages enter allowable install sets
- Hardening GitHub Actions secrets boundaries

## Lessons Learned
This incident highlighted several areas where we can strengthen our overall supply-chain security posture. In particular, it showed that relying on a single publishing token carries inherent risk, and that our release process can benefit from additional layers of protection and verification.

We have already begun implementing improvements designed to make our publishing process more resilient, including expanded credential controls, better separation across publishing workflows, and more robust configuration of our build and deployment tooling.

These improvements will meaningfully reduce the likelihood and blast radius of similar supply-chain attacks.
