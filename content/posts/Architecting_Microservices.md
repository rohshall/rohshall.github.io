+++
date = '2025-06-19T17:52:28-07:00'
draft = false
title = 'Architecting_Microservices'
+++
# Architecting Microservices


## Microservices architectural anti-patterns

Don't make services glorified databases:
https://www.confluent.io/blog/data-dichotomy-rethinking-the-way-we-treat-data-and-services/
Also, tight coupling between services indicate that we have a distributed monolith.



## Monitoring

Things that should be monitored in a microservice include: 

1. availability
2. performance
3. response time
4. latency
5. error rate
6. application logs

These metrics are collected by an event-management system, which correlates all the data from feeds like service monitoring, log monitoring, and infrastructure monitoring, and provide actionable alerts when something happens. “You absolutely need to have monitoring pieces put in place from day one so you understand what is happening within that distributed environment, those distributed instances and how they work together,” said Berg.



## Microservices management patterns:

Microservice management practices can come from the design patterns of the architecture itself. 


## Centralized configuration

Centralized configuration allows developers to deploy across multiple different environments without having to worry about the infrastructure (as in, specifying the configuration for each infrastructure). 



## Service communication patterns

Two most important considerations for service-to-service communication:

1. Service discovery
2. Handling service failure - Circuit breaker

### Service discovery

Design patterns like service discovery can automatically find new services that become available in the platform so developers don’t have to worry about it.

### Circuit Breaker

And a circuit breaker pattern can safeguard against unpredictability. For instance, if you have a microservice that depends on another microservice, but can’t guarantee that microservice is always available, you can provide an alternate path to a different service. “A good example would be the Amazon homepage. If a customer goes to buy a book, but the recommendation service is currently down, it shouldn’t stop people from buying the book,” said Crowther. “If you have a graceful way of handling that outage on the recommendation engine without disrupting the rest of the flow, then that is a good design pattern to implement because at least you are giving 95 percent of the functionality back to the customer.”

### Service mesh

This is handled typically by a service communication libraries like Twitter's Finagle.
But, when we have legacy services, it is difficult to change them to use the libraries.
So, the solution was to use service mesh - set of proxies that handle the service communication
http://philcalcado.com/2017/08/03/pattern_service_mesh.html



## Running transactions across microservices

Alternative to distributed transactions:
https://initiate.andela.com/event-sourcing-and-cqrs-a-look-at-kafka-e0c1b90d17d8
At Andela, each kafka topic maps to a bounded context, which in turn maps to a microservice.


## What are aggregates

Aggregates are stateful entities which are referred by their IDs by other aggregates/objects. So, aggregates can be put in any service. Also, the rule is a transaction can update only one aggregate, so a transaction remains limited to one service.
https://www.infoq.com/articles/microservices-aggregates-events-cqrs-part-2-richardson
A distinctive benefit of this approach is that the aggregates are loosely coupled building blocks. They can be deployed as a monolith or as a set of services. At the start of a project you could use a monolithic architecture. Later, as the size of the application and the development team grows, you can then easily switch to a microservices architecture.
The role of the Aggregate is to group all required details (small data changes) in order to have one valid business state change (from the domain point of view). The aggregate might be implemented as a class (state and behavior), but its value in design is the fact that it represents only one business change operation, regardless of how many details needs to be changed. That's why the aggregate defines a consistency boundary, every change inside that boundary is part of that unit of work. And it's always about domain state changes, not about data itself (I'll provide an example later).
Moreover, that's why you don't have aggregates involved in the same unit of work. It doesn't make sense, since each aggregate represents a whole immediately consistent operation that results in one business state change. That change is represented in DDD as a Domain Event. From a high level point of view it can end here, but if we go lower, we need to handle how things will be persisted. However, this is an implementation detail, outside of DDD.
From Persistence point of view you can store the changes themselves (Event Store) and/or as projections i.e actual data that can be queried easily. That data model is unrelated to the Domain Model, it exists as an implementation detail to have fast(er) reads.
You may think "ok, behaviour first, but we do need input value or existing data in order to perform a domain operation". That's true, however that data is transient and always dependent on the functionality that needs it. When modelling some Domain functionality, our focus is on the rules first and then whatever data is needed (both as input and output).
http://blog.sapiensworks.com/post/2018/01/08/DDD-Aggregate-groups-behaviour-not-data

### Aggregate always means command model.

An aggregate defines consistency boundaries, that is, everything inside it needs to be immediate consistent. This is important, because it tells us that no matter how many actual changes (state mutations) need to be performed, we have to see them as one commit, one unit of work, basically one 'big' change made up from smaller related changes which need to succeed together.
An aggregate instance communicates that everything is ok for a specific business state change to happen. And, yes, we need to persist the busines state changes. But that doesn't mean the aggregate itself needs to be persisted (a possible implementation detail). Remember that the aggregate is just a construct to organize business rules, it's not a meant to be a representation of state.
So, if the aggregate is not the change itself, what is it? The change is expressed as one or more relevant Domain Events that are generated by the aggregate. And those need to be recorded (persisted) and applied (interpreted). When we apply an event we "process" the business implications of it. This means some value has changed or a business scenario can be triggered. More about that in a future post.
You can say the job of the aggregate goes like this: "Based on the input you gave me and business rules that I know, the following business state changes took place: X happened with these details. Do whatever you want with it, it's not my job, I'm done here".
http://blog.sapiensworks.com/post/2016/07/14/DDD-Aggregate-Decoded-1
http://blog.sapiensworks.com/post/2016/07/14/DDD-Aggregate-Decoded-2
Why eventual consistency is in fact better than consistency:
http://blog.sapiensworks.com/post/2018/01/22/Why-do-we-need-eventual-consistency
http://blog.sapiensworks.com/post/2016/07/23/DDD-Eventual-Consistency
What is Bounded context?
http://blog.sapiensworks.com/post/2014/10/31/DDD-Identifying-Bounded-Contexts-and-Aggregates-Entities-and-Value-Objects.aspx
I'm not talking about the fact they're in different offices. They have their own internal organization, their own internal rules, employees etc. John doesn't go into Rita's office and modifies the payroll and Rita doesn't go to John's and modifies his code. They could but it would be such a scandal, because if they do that they would be overstepping their boundaries. If Rita finds a bug in the accounting software (developed in-house) she calls the IT department to handle it. She doesn't fire up Visual Studio and starts messing with the code. It isn't her responsibility and she doesn't know how to do it anyway, even if she knows that VS is the program used by John to write code. In fact, VS would be a very strange software on an accountant's computer. Like wise, the payroll files or invoices have no place in the IT department.
http://blog.sapiensworks.com/post/2012/04/17/DDD-The-Bounded-Context-Explained.aspx



## Practical implementation of microservices:

Kafka could be the event stream through which microservices communicate
https://www.confluent.io/blog/apache-kafka-for-service-architectures/
http://www.devramble.com/microservices-with-spring-boot-spring-cloud/

Are REST calls the best way to communicate among services?
https://capgemini.github.io/architecture/is-rest-best-microservices/

