---
id: tp-public-api-access-limits
title: Threat Vault API Access Limits
sidebar_label: Threat Vault API Access Limits
---

The Threat Vault API included with the Threat Prevention and Advanced Threat Prevention security subscriptions use a rate limiter to throttle the maximum number of API calls that a user can make in both a 1 minute and a 24 hour period. The API usage is throttled based on a sliding window rate limiter, whereby any given 1 minute and 24 hour period allows for a maximum of 200 and 2,000 requests, respectively. This means your API usage count is incremented by one for every request made and decremented by one after both a 1 minute and a 24 hour cycle (for their respective rate limiters)  - in other words, the API does not have a hard reset time, but instead progressively regains capacity at the equivalent rate at which it was consumed.

Consider the following example:

A user makes 100 concurrent requests hourly from 1:00PM to 6:00PM:

   - The timestamps associated with a request is recorded and tracked until it expires for *both* the 1 minute and 24 hour period rate limiters.
   - In this case, the 1 minute rate limiter for each batch of concurrent requests deducts from your maximum query limit for 1 minute after the requests are made. This means your rate limit was consumed by 100 for that one minute period (the maximum is 200), but is refreshed when the timestamp for a request expires, one minute after the requests are submitted.
   - Likewise, the 24 hour rate limiter for each batch of concurrent requests deducts from the 24 hour max query limit of 2,000 on an hourly basis from 1:00PM to 6:00PM. This means your max query of 2,000 drops to 1,900 at 1:00PM, 1,800 at 2:00PM, 1,700 at 3:00PM and so on until 6:00PM, when the final batch of requests is made, resulting in a total consumption of 600 requests (1,400 remaining). Your 24 hour rate limit will be reduced until the following day, at which point, at 1:00PM the timestamp for the first batch of 100 concurrent queries that you made the previous day expires, returns a capacity of 100 requests, raising your total to 1,500. An hour later, the second batch of concurrent requests made the previous day ar 2:00PM expires, returning another 100 requests to your maximum capacity (raising it to 1,600). This occurs until 6:00PM, when the last batch of concurrent requests were made. Upon expiration of this batch, your maximum capacity returns to 2,000.

If you exceed the usage limits, the API response returns the following code: 429, indicating that too many requests have been made. You can monitor your usage status by referring to the following header responses:

| Header Response                          |  Description                       |
| ------------------------------------     | ---------------------------------  |
| `X-Minute-RateLimit-Limit`               | Specifies the maximum number of requests that the user is permitted to make in (one minute). |
| `X-Minute-RateLimit-Remaining`           | Specifies the number of requests remaining in the current rate limit window (one minute) |
| `X-Minute-RateLimit-Reset`               | Indicates the timestamp at which the current rate limit window (one minute) resets. |
| `X-Day-RateLimit-Limit`                  | Specifies the maximum number of requests that the user is permitted to make in (one day). |
| `X-Day-RateLimit-Remaining`              | Specifies the number of requests remaining in the current rate limit window (one day). |
| `X-Day-RateLimit-Reset`                  | Indicates the timestamp at which the current rate limit window (one day) resets. |
