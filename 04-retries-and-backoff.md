# Retries and Backoff

A retry is the automatic reprocessing of a failed job, and backoff is the deliberate, growing delay inserted between attempts so you do not hammer a struggling dependency. They matter because most job failures are transient (a timed-out request, a momentarily unavailable database, a rate limit) and will succeed if tried again a moment later. Retrying immediately and forever, though, turns a brief outage into a self-inflicted denial-of-service against your own downstream systems. The art is retrying enough to ride out blips while backing off enough to let things recover.

## Exponential backoff

Increase the delay between retries exponentially rather than retrying at a fixed interval. A typical schedule waits 1 second, then 2, then 4, then 8, then 16, doubling each time up to a cap. This gives a failing dependency progressively more room to recover, and it prevents a thundering herd of synchronized retries from all clients hitting at once. Pair it with a maximum attempt count and a maximum delay ceiling so a permanently broken job does not retry forever or wait for hours between attempts.

## Jitter

Add randomness to backoff delays so retries spread out instead of synchronizing. Without jitter, many jobs that failed at the same instant (for example during a shared outage) will all retry at exactly the same future moments, recreating the spike that caused the failure. Jitter perturbs each delay by a random amount, so retries arrive smeared across a window. A common form is "full jitter," where the actual wait is a random value between zero and the computed exponential delay, which spreads load most evenly.

## Knowing when not to retry

Distinguish transient failures, which should be retried, from permanent ones, which should not. A 503 or a connection timeout is worth retrying; a 400 validation error, a 401, or a malformed payload will fail identically every time and just waste attempts. Classify errors and fail fast on the permanent ones, sending them straight to a dead-letter queue instead of cycling through the retry schedule. Also cap total attempts, because a job that has failed many times is usually telling you about a real bug, not a passing blip.
