
Internal post-mortem: https://github.com/PostHog/incidents-analysis/pull/58

On 28th February, PostHog Feature Flags went down for about 3 hours, from 15:30 to 18:10 UTC.

Our database started stalling, which caused a small blip in service. Unfortunately, we then proceeded to DDOS ourselves as all clients tried to retry feature flags. This problem never showed up before because we had sufficient capacity to handle 5x the peak load. As flags have grown, this unfortunately isn’t true anymore. To further exacerbate this problem, our failover healthchecks for the database all returned healthy, so we kept slowly trying to service all requests.

As a result, some users had their flags fallback to empty responses, and some had their applications crash because of exceedingly long timeouts. However, users using local evaluation were not affected. This is again our bad as SDKs were missing timeout configurations specific to feature flags - this ought to generally be much lower than regular capture requests.

We spent way too much time trying to figure out what was wrong, which is why this incident lasted so long - this is completely unacceptable and we’ll be making several changes to make sure this doesn’t happen again.

To start with, all SDKs will stop retrying feature flag requests, and have customisable timeouts specific for flags. Further, the default here will be 3 seconds, compared to the 10 second default for all other requests. This is already released for all SDKs now. This should make sure that even if something unexpected goes wrong with our servers in the future, your application never goes down.

Secondly, we’ll add a manual override to our db healthchecks, so we can much more quickly respond to issues like these, and bring flags back up.

As a more long term solution (Q2 2024), we’ll move out feature flag requests from our Django monolith as we hit these scaling limits, and move to a more scalable system focused just on feature flag reliability.



