# Why Background Jobs

A background job is work that runs asynchronously, outside the request that created it, so the caller gets a fast response while the slow or risky work happens elsewhere. The reason to use one is that synchronous request handling ties the user's wait time and the request's success to the slowest dependency in the chain. By moving that work into a job, you return an acknowledgment in milliseconds and let a worker handle the heavy part on its own schedule. This separation is the foundation of resilient, scalable systems.

## Latency isolation

Move slow work off the request path so the user is not blocked by it. Sending an email, resizing an image, calling a third-party payment API, or generating a report can take seconds, and a synchronous handler forces the user to wait for all of it. With a background job, the handler validates input, enqueues the work, and returns a 202 Accepted or a created-resource response right away. The perceived latency drops to the cost of writing one row or one queue message, while the actual work proceeds out of band.

## Load smoothing and backpressure

Use a queue to absorb traffic bursts that would otherwise overwhelm a downstream system. When a spike of requests arrives, a synchronous system has to process every one immediately or fail, but a queue lets the work accumulate and be drained at a sustainable rate. Workers pull jobs as fast as they can handle them, which naturally rate-limits pressure on databases and external APIs. This is backpressure: the buffer grows under load instead of the system collapsing, and you can scale worker count to match demand.

## Failure isolation and retry

Background jobs let failed work survive and be retried instead of being lost. In a synchronous request, a transient error (a timed-out API, a brief network blip) usually means the whole operation fails and the user sees an error. A job sits in durable storage, so when it fails the system can retry it later without the user being involved at all. This decoupling also contains blast radius: if the email provider is down, emails queue up and send when it recovers, while the rest of the application keeps serving requests normally.
