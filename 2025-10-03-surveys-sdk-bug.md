# PostHog Surveys SDK Bug - October 3, 2025

## Summary

A bug in the PostHog Surveys SDK caused anyone in an older SDK version (<) to have increased number of exceptions.

## Timeline

Times in UTC.

- 10:45: we deployed the new SDK version (1.270.0) that has the issue.
- 14:59: [a support engineer notices two similar reports in tickets](https://posthog.slack.com/archives/C075D3C5HST/p1759503579040139)
- 15:25: we start the process of reverting the potential PRs at fault [PR 1](https://github.com/PostHog/posthog/pull/39108/files) [PR 2](https://github.com/PostHog/posthog-js/pull/2397)
- 16:00: [Error Tracking team notices the throughput of exceptions is up](https://posthog.slack.com/archives/C087FAT5FK5/p1759507228619759)
- 16:11: PRs are now reverted, issue is no longer happening.
- 16:25: [kick off the incident](https://posthog.slack.com/archives/C09JR5WV0JG/p1759508704952599)
- 22:28: we resolve the incident and start the documentation phase.

Overall duration until mitigation: 5h26min.

## Root Cause Analysis

This is [the culprit PR](https://github.com/PostHog/posthog-js/pull/2355) that introduced the issue.

It introduced a new check in the surveys SDK that relied on a function that was only available in newer versions of the SDK (>= 1.257.1).

When it was merged, both code in the surveys SDK (posthog-surveys.ts) and in the extensions (extensions/surveys.tsx) relied on this function.

It's not a problem for the SDK. However, since the extensions are downloaded async and always on the most updated version, it caused an error where the function was not available.

This ended up causing an increased number of exceptions for anyone using an older version older than 1.257.1.

## Impact

305 teams affected. We were able to preemptively reduce this to 90 because [we found a way to exclude exceptions caused by this issue](https://github.com/PostHog/posthog/pull/39126).

However, there were also other errors caused by this, which is why we still have 90 customers affected.

## Remediation

We reverted the PRs and released a new version of the SDK (1.270.1) that fixes the issue by reverting the changes.

### Immediate Actions

- Refund any customers that got an increased error tracking bill because of the higher number of exceptions.

### Short-term Improvements

- Make it so the function `isDisabled` on the `posthog-persistence.ts` is nullable - as it's not always available.

### Long-term Improvements

- Add tests the SDK that uses older versions of the SDK. This way, we can guarantee we'll not merge PR that breaks the SDK for older versions.
