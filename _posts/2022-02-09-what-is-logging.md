# What is logging?

Logging is printing out whatever happening in your program. **The End**.

If everything could be that simple! There is a whole theory behind it, lots of pitfalls to avoid and best practices 
to follow. This is the first article in a series dedicated to logging, which will introduce you to the topic and explain
why you need it.

## Observability
Logging is one of the so called "three pillars" of observability. Observability is a term from control theory 
describing how well internal states of a system can be inferred from knowledge of its external outputs 
(see [Wikipedia](https://en.wikipedia.org/wiki/Observability)).

Three pillars of observability are:
- Logging
- Metrics
- Distributed tracing

### Logging
Logs are timestamped and immutable records of application events. The classic way of doing it is to output text 
statements into a file stored somewhere on the server. Nowadays with containers, microservices architecture and 
cloud infrastructure logs are collected and stored in centralized space using tools like [ELK](https://www.elastic.co/what-is/elk-stack) or [Datadog](https://www.datadoghq.com/).

### Metrics
Metrics are numeric values calculated or aggregated over a period of time. Metrics are collected from variety of 
sources such as CPU, networking, databases and services. Metrics have a number of uses, most popular of which are 
alerting and automatic scaling of cloud services based on number of requests per second or resource usage. 

### Distributed tracing
Distributed tracing is a method of following transaction or request through the system. It's an extra information, 
for example request id, attached to each log record and propagated through the system. It allows to separate the 
journey of one particular request through the system from other requests running in parallel.  

These tools together help developers, admins and IT support to resolve bugs, troubleshoot client requests and 
prevent failures before they manifest themselves.

## Why to use it?
It may all sound overly complicated, but there is a real problem solved. For small enough program with no branching 
logic and output easily deductible from input it may not make much sense. But imagine a system where output depends 
on environment configuration and state, internal and external, changing from request to request. Even reproducing an 
issue may become a problem by itself.

There are lots of situations where developer won't have access to the runtime of the application. Examples are: 
mobile and embedded applications, software run on client's premise, other restrictions including security 
and compliance.

In many languages program terminates with stack trace pointing to exact point of failure. But very often what also 
required to resolve the issue is knowing the input and state of the system just prior the error. 

## How to start?

Don't delay it. Start logging right now. You don't have to master it to get benefits from using it. Most languages come 
with some sort of logging out of the box.

## Where to go next?




Three pillars: traceability, ...
Use cases, real life situations
How to start or where to start
Conventions - team agreement
Ever changing goal