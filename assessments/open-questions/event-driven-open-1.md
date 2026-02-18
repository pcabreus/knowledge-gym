Questions:
Q1) Define an "event" in the context of event-driven architecture and explain how it differs from a command.
Q2) Given a checkout → payment → order update flow, draw the producers, topics/queues, and consumers, and explain how you would ensure at-least-once processing without double side-effects.
Q3) What is the Outbox pattern and why does it solve the dual-write problem? Sketch a simple implementation outline.
Q4) Explain Dead-Letter Queues (DLQs): when to use them, how to surface poison messages, and how to triage them.
Q5) Discuss strategies for schema evolution of events and how you would coordinate producer and consumer changes in production.
Q6) How do you handle backpressure between fast producers and slower consumers? List at least three practical mechanisms.
Q7) Describe idempotency in event handlers: common implementations, storage choices, and how to handle long-lived deduplication state.
Q8) Trap question: If you replay historical events into a live consumer that also receives new events, what issues might arise and how do you mitigate them?
