# Logging pitfalls

In this article I'll cover some pitfalls on your way to master logging. I'll list them in no particular order with 
short explanation why I consider it a bad practice and references to best practices around it.

## Not logging
How big should be your program to get benefit of logging? I don't think it's a matter of size. In my experience I added 
logging to one file scripts and one line methods. It's not about some magic ratio, like 1 logging statement per 10 lines
of code. It's about business logic of your application. From logs you should be able to understand when, what and how 
has happened. The flow and the outcome of particular request or transaction.

## Using `print` for logging
Producing log messages can be accomplished using bare `print` statements. And it still might be fine in rare cases. But 
most of the time that's not enough. Log record has a number of traits each important to have:
- timestamp
- level
- context

Once your program becomes somewhat related to host or network resources, timestamps are necessary. With timestamps 
you can do a number of things:
- match HTTP request timeout to network downtime
- match slow application performance to database CPU usage spike
- combine logs from multiple services into ordered list of events happened in your system

Log level allows to control detalization of logging without changing any code. Popular practices are:
- having DEBUG level in development environment and INFO in production
- using dynamic logging configuration to enable DEBUG or TRACE level logging in production when unreproducible bug 
arises

Many programs nowadays either use multithreaded or asynchronous programming. Mixing and matching different log records 
is impossible without knowing the context of those records. Log records can contain extra information such as 
file name, class name, thread name, request or user id. They allow to filter out records to get a slice of events 
happened in particular context. Distributed tracing solves the problem of matching logs from different services in 
microservices architecture.

## Logging too much
When people get fancy about logging or told to do logging by their team without much explanation they may not have 
a clue of how much logging to do. From logs you should be able to understand what's happening in your program. Does 
that mean you have to add logging after every line of code? What if you iterate through collection of items? Should you 
log something on each iteration or only before and after iteration was done?

Those are all good questions and there is no definite answer to them. Log levels somewhat help with this. At INFO level 
you probably want to log outcome of your requests or transactions. At finier levels you may log intermediate stages of 
some complex algorithm or process. Then during development you can enable excessive logging to make troubleshooting 
easier. In production though because of the volume of requests or security consideration you may have to go with less 
verbose levels of logging. This is not an exact science. You'll have to figure it out yourself for your team given 
the constraints and requirements of your project.

## Logging too little
Logging too little is often worse than logging too much. You can reduce log noise by tuning logging level configuration. 
But you can't do anything if there is no logging statements to enable. Solving lack of logging will always require 
a code change.

When writing code if you have to debug through some code to ensure it works or to fix failing test, that's a good hint 
that more logging should be added. If you think it doesn't bring much value, make it log on DEBUG or TRACE level. 
Or ask opinion of your colleagues. Most of the times people who are not familiar with your code would prefer to be able 
to see something in logs, rather than nothing. You'll probably thank yourself when you return to that piece of code 
in few months time completely blank.

## Silencing Exception

TODO

## Ever changing goal

It's a constant learning process based on feedback from your support team and production issues troubleshooting. 
Sometimes you'll have to add more logging to the pieces of code causing trouble. In other case some logging will have 
to be switched off because of being too noisy.

