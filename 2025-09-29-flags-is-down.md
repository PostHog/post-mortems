# PostHog Feature Flags Service Outage - September 29, 2025

Internal post-mortem: <https://github.com/PostHog/incidents-analysis/pull/120>

On September 29th, PostHog Feature Flags went down for about 1 hour and 48 minutes, from 16:58 to 18:46 UTC, affecting approximately 78% of flag evaluation requests in the US region.

## What Happened

Our writer database experienced high load from person ingestion, which caused initial connection issues. Unfortunately, we had just deployed a change that reduced our database connection timeout from 1 second to 300 milliseconds. This created a perfect storm: pods couldn't connect to the database fast enough, triggering automatic retries that amplified the problem into a full service outage.

The situation was made worse by our retry logic, which essentially created a self-inflicted DDoS attack. Pods that crashed continued receiving traffic for up to 45 minutes, and our standard rollback procedures failed due to configuration issues. What should have been a 5-minute fix turned into nearly 2 hours because timeout values were hardcoded and required full deployments to change.

As a result, most users experienced 504 errors when trying to evaluate feature flags. The ironic part? Most of these failing requests didn't even need writer database access—they failed simply because they hit pods stuck in crash loops.

## What We're Doing About It

We've already completed several immediate fixes:

- Made all database connection settings configurable without code changes
- Increased timeout values to handle load appropriately

Here's what we're implementing to prevent this from happening again:

**Short term (this sprint):**

- **Decoupling reader and writer operations** - Most flag evaluations are read-only and shouldn't fail when the writer database has issues. We're separating these paths so only flags that actually need write access are affected by writer DB problems.
- **Fixing our rollback procedures** - We're updating our ArgoCD rollback runbook with clear, step-by-step instructions and making it easily accessible from all alerts.
- **Implementing circuit breakers** - We're adding proper circuit breakers with exponential backoff to prevent retry storms from amplifying problems.
- **Aggressive health checks** - Pods that fail will be immediately removed from traffic rotation, not after 45+ minutes.

**Longer term (Q4 2025/Q1 2026):**

- **Improved tooling** for identifying and terminating unhealthy pods during incidents
- **Load testing** to understand connection behavior under various contention scenarios
- **Quarterly drills** to practice emergency procedures like rollbacks

## The Bottom Line

This outage was preventable and lasted far longer than it should have. The combination of hardcoded values, retry amplification, and confusion during recovery turned a minor database issue into a major service disruption.

We're treating this as a critical learning opportunity. The architectural improvements we're making—especially decoupling read and write operations—will fundamentally improve the resilience of our feature flags service. Combined with better operational tooling and procedures, we're confident these changes will prevent similar incidents in the future.

We apologize for the disruption this caused. Feature flags are critical infrastructure for many of our users, and we take this responsibility seriously. The improvements we're implementing will make the service more robust and ensure that even when individual components fail, your feature flags keep working.
