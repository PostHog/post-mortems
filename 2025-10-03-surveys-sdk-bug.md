# PostHog Surveys SDK Bug - October 3, 2025

## Summary

A bug in the PostHog Surveys SDK caused anyone in an older SDK version (< 1.257.1) to have increased number of exceptions.

## Timeline

Times in UTC.

- 10:45: we deployed the new SDK version (1.270.0) that has the issue.
- 14:59: [a support engineer notices two similar reports in tickets](https://posthog.slack.com/archives/C075D3C5HST/p1759503579040139).
- 15:25: Issue is confirmed by the surveys team. We start the process of reverting the potential PRs at fault [PR 1](https://github.com/PostHog/posthog/pull/39108/files) [PR 2](https://github.com/PostHog/posthog-js/pull/2397)
- 16:00: [Error Tracking team notices the throughput of exceptions is up](https://posthog.slack.com/archives/C087FAT5FK5/p1759507228619759)
- 16:11: PRs are now reverted, issue is no longer happening.
- 16:25: [kick off the incident](https://posthog.slack.com/archives/C09JR5WV0JG/p1759508704952599)
- 22:28: we resolve the incident and start the documentation phase.

Overall duration until mitigation: 5h26min.

## Root Cause Analysis

This is [the culprit PR](https://github.com/PostHog/posthog-js/pull/2355) that introduced the issue.

The PR was about adding a new check in the surveys SDK to use the `posthog.persistence` instead of `localStorage` directly. However, to ensure backwards compatibility, we also needed to check if the `posthog.persistence` was available.

For that, we used the `isDisabled` function from the `PostHogPersistence` class. And added a utility in the `survey-utils.ts` file to check if persistence was available.

But, that function was only added on a [PR made on July 11](https://github.com/PostHog/posthog-js/pull/2082), and only made available since the version 1.257.1 of the SDK.

When it was merged, both code in the surveys SDK (posthog-surveys.ts) and in the extensions (extensions/surveys.tsx) relied on this function.

It's not a problem for the code in the main SDK file (posthog-surveys.ts). However, since the extensions are downloaded async and always on the most updated version, it caused an error where the function was not available.

This ended up causing an increased number of exceptions for anyone using an older version older than 1.257.1.

It wasn't caught in the code review as we don't usually check when an API is added to the main SDK files.

## Impact

This issue only affected customers on older versions of the SDK (< 1.257.1). In total, 305 teams were affected.

It caused both any Surveys functionality to not work properly, as well as an increased number of exceptions in our customers' applications.

Regarding the Error Tracking bill, we were able to reduce this to 90 because [we found a way to exclude most of the exceptions caused by this issue](https://github.com/PostHog/posthog/pull/39126).

## Remediation

We reverted the PRs and released a new version of the SDK (1.270.1) that fixes the issue by reverting the changes.

### Immediate Actions

- Refund any customers that got an increased error tracking bill because of the higher number of exceptions. The refunds are already issued.
- Start incidents earlier. We should have started the incident as soon as we have noticed the issue (around ~14:59). Not almost two hours after.

### Short-term Improvements

- Make it so the function `isDisabled` on the `posthog-persistence.ts` is nullable - as it's not always available. Owner: @lucasheriques

### Long-term Improvements

- Add tests the SDK that uses older versions of the SDK. This way, we can guarantee we'll not merge PR that breaks the SDK for older versions. Owner: @lucasheriques
- See if there's a way (maybe a linter) to prevent adding new APIs to the main SDK files (posthog-\*.ts) that are not marked as potentially undefined. As they might cause breaking changes if used in an extension. Owner: @lucasheriques
