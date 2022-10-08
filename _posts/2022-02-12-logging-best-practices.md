# Logging best practices

This is a collection of generic best practices for backend logging. By no means each of the items contains 
exhaustive description. Many items deserve articles on their own which you can find on the Internet. This 
article is more like a starting point referencing directions worth exploring. 

## Centralized logging
In old days product used to be implemented as a monolith, single program, running on big server, talking to even 
bigger database and serving client requests. Logs usually were written into file(s) on the same server. To 
troubleshoot an issue developer had to look into those files, always stored at the same big server and the same 
location.

Cloud infrastructure and microservices architecture changed the landscape of service development. Containers may go 
up and down without human intervention, system logic is split between multiple services running on different hosts 
and working together to fulfill user request. It might be hard to find particular instance of service which served 
specific user request. Service instance may also be long gone because load reduced or service was updated. Trying 
to mix and match log records from different hosts in this highly dynamic environment is almost impossible.

Centralized logging emerged as an answer to this problem. Solutions like ELK, Datadog, New Relic etc. have emerged. 
They allow you to save logs from all your containers into single storage. Index it. And provide reach visualization 
and filtering capabilities. Using distributed tracing you can observe all relevant logs from different service in 
one single view.

## Structured logging
Traditionally log message was a human-readable text describing what happened and providing some context such as user 
email, transaction id or product name. As the volume of logs increased it became hard to navigate. Two different 
log records may relate to the same context but to understand that human have to read and match them. Example:

    ID 123 assigned to item "Umbrella"
    State changed to "ACTIVE" for id 123

In theory developer can filter these records using exact or pattern-matching. In reality similar messages from 
different contexts will be returned and developer will always have to fallback to eye-matching records to ensure 
they all talk about same thing.

Structured logging recognizes the importance of context and makes it a full member of log record. In structured 
logs example above might look like this:

    Item ID assigned, item_id=123, item_name="Umbrella"
    Item state changed, item_id=123, new_state="ACTIVE"

This approach makes context of the log record explicit, easy to consume and filter by it. Logs become not only 
human but also machine-readable. This allows to build on top of logs things like alerting, reporting and 
derive insights. 

## Correct log level
Every logging frameworks comes with the concept of log level or severity. It's an indicator of how important 
particular log record is. Log level allows to filter out important records from unimportant, e.g. errors from 
regular information messages. There is no single standard on log level naming. More importantly there is no 
single meaning of each log level. It will greatly depend on kind of your application. Inability to read 
configuration file for desktop application might be a warning, because it still can run from a clean slate. But 
for backend application that will be a fatal error, because it won't know what database to connect to.

Although different most logging frameworks will go from the highest priority level such as ERROR to the lowest 
like TRACE. Some frameworks allow to define your own log levels, what is useful if standard levels are not 
enough. If you think about some generic SaaS application log level breakdown might look like this:
- FATAL   - critical functionality is inoperable, all users of one or more clients are affected
- ERROR   - critical functionality is unavailable to one or a few users not affecting the majority of users.
- WARNING - non-critical functionality is unavailable
- INFO    - operation execution details, necessary for troubleshooting
- DEBUG   - non-essential implementation details, useful for troubleshooting
- TRACE   - algorithm or method implementation details, useful for development and rarely for troubleshooting

Your alerting setup will probably affect your definition of FATAL/ERROR levels. WARNING is often used to indicate 
some kind of development or operation technical debt. INFO is the default log level usually used in production. 
From this level you should be able to reason about your system. DEBUG is usually enabled in development environment 
for quicker troubleshooting of new features. Many logging frameworks allow dynamic config reloading. If specific 
peace of code starts failing in unreproducible way this allows to enable more detailed logging for it.

## Internal guidelines
Microservices architecture gave us the way to scale out development by splitting it to teams operating 
independently. But some things still have to be agreed between teams. Logging is one of those things. You don't 
want to learn intricacies between different teams approaches to logging in the middle of severe downtime. 
When you look at request going through different services developed by different teams you want to have a clear and 
consistent picture of what's going on. This is why documenting your logging approach is important.

## Internal libraries
Agreeing on logging best practices is only part of the deal. More often than not standard logging libraries don't 
cover all the details for you. Starting from as small as spelling that tracing id key logged with every message. Is 
it `traceId` or `TRACING_ID` or `myCompanyTracingId`? Internal logging libraries can not only enforce agreed 
conventions but also help everyone implement them. You may have more than one cross-cutting concern: request id, 
user id, client id, transaction id etc. You may have helpers around logging, configuring and formatting log records. 
It all takes time to implement and keep up to date with changing conventions. Integration with different frameworks 
and technologies pose their problems for logging too. If you treat logging as part of your infrastructure, because 
it's universal and doesn't depend on service business logic, then it makes sense to provide developers ready made 
internal libraries to use and allow them to focus on business features.

Small tip when implementing your library: look into fluent logging to make it more readable and extensible.

## Context
Tracing is often mentioned as a way to follow request processing through multiple microservices. The way it's 
implemented usually is: request id generated at API Gateway or other service exposed to outside world and receiving 
client's request. That request id is then attached to every log record emitted, passed over queues or RPC calls 
to other services to be logged or passed again. This allows developer to filter only log records related to 
particular API call. But this is only one of perspectives we can have on the logs.

Each log record comes with a context. And there are multiple ways you can slice and dice that context to get 
more information from your logs. Let's look at the example operation: customer added item X to their basket. 
What extra information or context we may want to know about this operation aside from request id?
- customer id - perspective allowing you to see all actions done or related to that specific customer
- session id - all actions done by customer in that login session
- basket id - all actions done to that basket

Those all are contexts you want to see across all your services. So next time you think about tracing don't stop 
on thinking about request only. Think about other perspectives you may want to see your logs through. Add and 
propagate that information through your system. Make them part of your internal logging library.

## Correlation ID
One API request might result in number of internal requests to different services, sometimes calling same service 
multiple times or enqueuing same type of message more than once. Filtering for each of such request or message 
processing logs becomes impossible, because all of them come with the same id. Common solution to this is to 
generate new request id for each outgoing call while passing original one as correlation id. In this case you'll 
be able to filter logs either by each individual sub-request by request id or observer the whole flow by 
correlation id.

## Constants for log messages
You may already be using structured logging but there is one step farther. You may rightfully be tempted to produce 
better messages like this one:

    message="User Peter added Hat to basket", user_id=123, product_id=456, basket_id=789

It's best of both worlds: both human and machine-readable.

But there is one problem. In case you want to see all occurrences of this log message you'll have to figure out 
the pattern. It might look like this `User * added * to basket`. And only after applying that filter you realize 
it returns other kinds of messages too, for example:

    User Peter added discount buy-one-get-one to basket

And we started with already quite detailed pattern. You probably have that experience of going through a number 
of iterations trying to come up with right pattern to match only the messages you want to.

My way to sort this out is to split log message to machine-readable code and only human-readable description. 
Example above might be transformed like this:

    code="basket_product_added", description="User Peter added Hat to basket", user_id=123, product_id=456, basket_id=789

You can make description optional, only for cases where code not expressive enough. It's also good idea to come up 
with some pattern of you codes, so that you reduce the chances of duplicate. Another big advantage of this approach 
is ease of tying up alerting to specific messages.

## UTC timestamp

TODO

