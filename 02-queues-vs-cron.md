# Queues vs Cron

A queue is event-driven and processes work as soon as it is enqueued, while cron is time-driven and runs a fixed task on a fixed schedule regardless of whether there is work to do. The distinction matters because using the wrong one produces either latency you did not want or load spikes you did not plan for. Queues fit per-event work that should happen promptly; cron fits periodic, batch, or housekeeping work that is naturally tied to the clock. Many real systems use both, and they often work together.

## What cron is good at

Use cron for recurring work that is defined by time rather than by an event. Nightly database backups, hourly cache warming, daily report generation, and weekly cleanup of stale records all map cleanly onto a schedule. Cron is simple, requires no message broker, and is easy to reason about because it runs at known times. Its weaknesses are that it has no built-in retry, no concurrency control, and no awareness of how much work exists, so a job that overruns its interval can overlap with the next run and cause trouble.

## What queues are good at

Use a queue for work triggered by individual events that should be processed soon after they occur. When a user signs up, you enqueue a welcome email; when an order is placed, you enqueue fulfillment; each event becomes one job processed independently. Queues give you per-job retries, parallel workers, prioritization, and visibility into depth and throughput. The tradeoff is operational complexity: you now run a broker (such as Redis, RabbitMQ, or a managed service) and worker processes that must be deployed, monitored, and scaled.

## Combining the two

Cron and queues complement each other when a scheduled trigger needs to fan out into many independent units of work. A common pattern is a cron job that wakes up on a schedule, queries for records that need processing, and enqueues one message per record instead of doing the work inline. This keeps the scheduled task fast and lets the queue handle retries, concurrency, and load smoothing for the actual processing. The cron part answers "when," and the queue part answers "how reliably and how fast."
