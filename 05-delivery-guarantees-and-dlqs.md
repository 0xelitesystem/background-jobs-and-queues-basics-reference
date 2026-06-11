# Delivery Guarantees and Dead-Letter Queues

At-least-once delivery is the guarantee that every job will be processed one or more times but never zero times, which is the realistic default for production queue systems. It exists because the only way to guarantee a job is not lost is to keep redelivering it until the worker confirms success, and that confirmation can fail even when the work did not. The practical consequence is that you get reliability at the cost of possible duplicates, which is why idempotency is the required companion to this model. Dead-letter queues are where jobs go when retries are exhausted, turning silent failures into visible, inspectable ones.

## The three delivery models

There are three delivery semantics, and the middle one is what you almost always run. At-most-once delivers a job zero or one times: it never duplicates but can silently lose work, which is rarely acceptable. At-least-once delivers one or more times: it never loses work but can duplicate, so handlers must be idempotent. Exactly-once is the ideal of one-and-only-one delivery, but in distributed systems it is effectively achieved by doing at-least-once delivery plus idempotent processing, not by some magic that makes duplicates impossible at the transport layer.

## Why at-least-once wins

At-least-once is the default because losing work is usually far worse than repeating it. A dropped payment confirmation or a lost order is a real business failure, while a duplicate is a problem you can neutralize with an idempotency key. The mechanism is acknowledgment: the broker holds the message as "in flight" until the worker acks, and if the ack does not arrive within a visibility timeout, the message becomes available again for another worker. This is what produces both the safety and the duplicates, and you accept the second to get the first.

## Dead-letter queues

A dead-letter queue (DLQ) is a separate queue that receives jobs which have failed past their retry limit so they can be inspected instead of silently dropped or retried forever. When a job exhausts its attempts or fails a non-retryable check, the system routes it to the DLQ along with metadata about why it failed. This keeps the main queue healthy and gives you a place to alert on, debug, and selectively replay failures after a fix is deployed. Treat a growing DLQ as a signal: it is the system surfacing the jobs that need a human to look at them.
