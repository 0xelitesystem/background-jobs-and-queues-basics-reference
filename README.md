# Background Jobs and Queues Basics

This is a plain-language technical reference for background jobs and queues. It covers why work moves out of the request path, how queues differ from cron, why idempotency matters, how retries and backoff behave, and what delivery guarantees and dead-letter queues actually give you. The goal is to give a working developer the mental model and the vocabulary to build async processing that survives failures.

## The core idea

A background job is a unit of work that runs outside the request that triggered it, and a queue is a durable buffer that holds those jobs until a worker is ready to process them. You use them to decouple a fast acknowledgment from slow or unreliable work: the web process accepts the request, writes a job, and returns immediately, while a separate worker drains the queue at its own pace. This buys you latency isolation (the user does not wait on a slow API), load smoothing (bursts pile up in the queue instead of crashing the service), and retry-ability (a failed job stays in the system instead of vanishing). The cost is that the work is now eventually consistent and distributed, so you have to reason about ordering, duplication, and failure explicitly.

## What is inside

- [01-why-background-jobs.md](01-why-background-jobs.md) why work moves off the request path and what async actually buys you.
- [02-queues-vs-cron.md](02-queues-vs-cron.md) how event-driven queues differ from time-driven cron, and when each fits.
- [03-idempotency.md](03-idempotency.md) why repeated processing of the same job must be safe, and how to make it so.
- [04-retries-and-backoff.md](04-retries-and-backoff.md) how to retry failed jobs without overwhelming downstream systems.
- [05-delivery-guarantees-and-dlqs.md](05-delivery-guarantees-and-dlqs.md) at-least-once delivery, what it implies, and where dead-letter queues fit.

## The stance

Assume every job will run more than once and design for it, because at-least-once delivery is the realistic default and exactly-once is mostly a marketing claim. Prefer a real queue over a database polling loop or a cron-driven batch the moment work is triggered by events rather than by the clock. Treat the dead-letter queue as a first-class part of the system, not an afterthought, because the jobs that land there are the ones telling you something is broken.

## More

Part of a catalog of single-file browser tools and plain-language references, all MIT licensed and dependency-free: [0xelitesystem.github.io](https://0xelitesystem.github.io/). Built by [elitesystem.ai](https://elitesystem.ai).

## License

MIT. Copyright 0xelitesystem 2026.
