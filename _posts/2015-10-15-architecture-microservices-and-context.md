---
layout: post
title: Building Microservices on Contexts
date: 2015-10-15 01:18:27.000000000 +07:00
image: https://www.ykode.com/assets/posts/microservices_meme.jpg
categories:
- Architecture
tags:
- architecture
- microservices
status: draft
type: post
published: true
---

Technology space is now buzzing with the word "Microservices". People are talking about it, companies sell things
around it. Hackers and open source developers make tools for it. Microservices seems to be "the next big thing". Is
that true, or not? Is Microservices is just another snake oil which will die faster than
[SOAP](https://en.wikipedia.org/wiki/SOAP)? Does the language you use now define "scalability"? Is that anything to
do about Microservices and programming language you use? Those are the legitimate questions one may ask.

<!--more-->

## What The Heck Is Microservice?

![Microservices Buzz Meme](/assets/posts/microservices_meme.jpg)

I'll quote from my favourite author, Martin Fowler:

> Microservice is an approach to developing a single application as **a suite of small services**, each **running
> its own process** and communicating with _lightweight mechanism_, often an HTTP resources API. These services are
> built around **business capabilities** and **independently deployable** by _fully automated_ deployment
> machinery. There's a **bare minimum of centralised management** of those services, which may written in different
> programming languages and different storage. -- James Lewis and Martin Fowler

Uncle Bob's definition is pretty much self-explanatory. The app is made by composing several independent,
self contained - deployable parts that works together and communicates. 

People implement Microservices usually due to following reasons:

* Faster response on business needs. Improving agility. It open opportunity to pivot and change business in timely
  manner versus sticking out to the rule that has been set and interdependence between components usually found in
  a monolith.
* Enable better customer experience. To be better improve the customer experience. We want to reduce churn.
  Especially if you have a service that customer can switch to your competitor without any effort.
* Cost reduction. When you want to expand, Microservices enable many business rules added quickly by only adhering
  contracts between services, avoiding high cost refactoring.

Microservices allows more agility on the software design than the
[Monolith](http://martinfowler.com/bliki/MonolithFirst.html). It allows less coordination between team to achieve
results. Say, if we have identity service that manage user credentials and takes care about the security. In one
point of time we want to implement two-factor authentication. If we have separate service, it'll be faster to
modify the login service only and left the other services unmodified.

## Is Microservices Architecture For You

![Molecule Model](/assets/posts/molecule_model.jpg)

I found out that Microservices is only logical if you have fairly complex business rules. If you have very simple
CRUD business rules, Microservice Architecture only gives you another overhead on your operations. If you're
thinking about utilising Microservice Architecture, you need to take automation very seriously. Neglecting it is
considered [antipattern](https://en.wikipedia.org/wiki/Anti-pattern) and will incur more costs in the long run.

Because Automation is one of the inseparable [characteristics of
microservices](http://martinfowler.com/microservices/), building one requires a change of culture to
automated unit testing, automated deployment, configuration management, service discovery, and quality assurance.
Quality Assurance will take a huge role on doing unit, functional, and security testing during development.

If you cannot find a way to automate the testing and deployment, **don't do Microservices**. Microservice without
automation is waste of time and energy. You'll be fighting with the deployment complexities rather than getting
things done.

## How Microservices Are Built

![Connected Microservices](/assets/posts/microservices_connected.jpg)

There are two ways building a Microservice-based applications.

* Building a monolith first and separate components as it goes. 
* Building microservices from the beginning.

Either way, there will be a question on how we separate concerns to independent services. How small (or big) a
service is? 

Going back to the premise of Microservices, the separation of the services should be based on **Business
Capabilities**. This is where people are usually tripped on. Engineers mostly focused on technical cohesion rather
than functional cohesion in regards of reusability. They separate services in horizontally. Creating artificial
interdependence between services, and introducing a tightly coupled system when any changes will bubble down the
layer. There's no single owner of business capabilities, because it's spread too thin horizontally. This
potentially creating a situation when each layer focusing on protecting their workflow from "outsider" (i.e another
layer above or below), rather than focusing on delivering business capabilities. It defeats the purpose of having
multiple services to encourage agility and removing interdependence.

The easy way of avoiding the pitfall of horizontally dividing layers is to think each service as if it is the
product itself. Divide the whole product to small contexts. Each service is a representative of a specific domain.
If you ever study or implement [Domain Driven Design](https://en.wikipedia.org/wiki/Domain-driven_design), the
smallest unit is the **Bounded Context** to be able to be deployed as its own independent services.

### Layered Service Anti-Pattern

Let say we have an e-commerce website. If we divide the technical cohesion, we may end up with these set of
services:

![Layered AntiPattern](/assets/posts/layered_antipattern.png)

We have storage service to expose SQL to the repository service. Repository service provides interface and act as
data access layer which provides models to application service. Application Service expose all APIs to the
front-end services. 

Now we have delivery dependency whenever there's changes or improvements on business logic because there are a lot
of out of process calls happening. The separation of concern serves technical concern and no business value because
business logic is scattered throughout every layer. We end up with something that resembles monolith with a lot of
network calls. That's very bad because now we need to handle network edge cases which we don't need on a monolithic
application. This services architecture is not composable and only increase complexity without delivering any
value.

### Define Services By Context

If we divide the e-commerce business logic by contexts we may end up with following set of services:

![Context Based Services](/assets/posts/context_services.png)

Now we divide the software into services that serve a specific business capabilities. Each of them can have
different stack of technology, and be made by an independent team from storage up to the UI. It's advisable for
each of the service does not share code. Each individual service does not share common database schema because that
would mean tight coupling. If services share database schema, there will be a situation where you change database
schema and render your application completely useless.

Because each of the services is made by their own respective team, they'd need something to be able to communicate.
That's where the contract between services comes into play. It's advisable to use a standard protocol such as REST
and [Protocol Buffers](https://developers.google.com/protocol-buffers/) for communication with other services.

### No Shared Storage

Each type of service **cannot have shared storage**. For example, a product catalogue service cannot have access to
user storage. There's no assumption that the services will run in the same machine or in the same data centre. By
using contract mentioned above, a service can ask another service for some data they need. For example, an **Order
Service** may want to know the data of the user and the item bought. The service can ask the identity service for
the data it needs and product catalogue service to get the item price. It copies the required information and save
to its `Order` table. When the user do payment, it wil ask **Order Service** for the order information, get the
total price and execute the payment. **Payment Service** will then save the receipt to its own storage containing
some information it gets from the **Order Service**. 

Up until this point we discover some of the characteristic, or rule of thumb where we building or refactoring
Microservices:

{% newthought 'Vertical Decomposition' %}. The system is divided into specific vertical with its own team, storage, and
  technology stack. It serves one business capabilities and domain.

{% newthought 'Shared Nothing' %}. There's no shared mutable state between services. There should no shared HTTP sessions.
  However, multiple instances of a service could share a database.

{% newthought 'Data Ownership' %}. For each context, there's only one service in charge. Other system could ask for read only
  access to the data provider using agreed protocol. Use the copy of data and extract the information needed to the
  service storage.

## Deployment and Scaling Up

Scaling up a microservices is actually pretty straightforward. Gather data as much as possible about the
characteristics of each services. In our example, **Product Catalogue** service there will be a lot of read access.
This way we may implement a cache strategy to increase the throughput of the service as well as using a database
that is optimised for read.

If we need to scale the service horizontally, we may put a load balancer to balance the load of each services.
Let's say we have a huge requests to our **Order Service**. We may spin up additional instance of **Order
Services**.

![Load Balanced](/assets/posts/load_balancer.png)

As you can see, many instances of services within _the same context_ can share database. It's so often that a
load-balanced applications do that. This set up, however, may lead to bottleneck due to shared write. This can be
done by scaling the storage too. In many cases, we don't need a strong consistency [even for banking
system](http://highscalability.com/blog/2013/5/1/myth-eric-brewer-on-why-banks-are-base-not-acid-availability.html).
In this case, an eventual consistency storage with better availability may be a wise choice. An architectural
pattern such as [Command-Query Responsibility Segregation](http://martinfowler.com/bliki/CQRS.html) and [Event
Sourcing](http://martinfowler.com/eaaDev/EventSourcing.html) maybe better suited for more scalable software
architecture.

### Deployment

![Continuous Integration](/assets/posts/Continuous-Integration.png)

A system consists of multiple small services written by agile team needs to be deployed in timely manner. Managing
a cluster of small services is not easy. This is a huge shift of the culture. A team that deploy a microservice
needs to have these:

**A disciplined coding practice**. Because in Microservices there are a lot of moving parts, a disciplined coding
  pratices is needed. Developers should be able to deliver a high quality, clean, and fully tested code.

**A complete test**. All kind of automated tests on every layer and environment. Team needs to be very
  disciplined on writing unit tests, all the way up to integration tests on every tier. For example we have three
environments: dev, staging, and production. For a code to be able to be promoted from dev to staging, it needs to
pass all the tests happening on dev. The higher risk the environment, the stricter the rule. For example, if dev
environment needs only to pass all the unit tests and integration tests. A staging environment needs to pass a
performance testing to be able to be promoted to production environment.

**An automated infrastructure**. Microservices with manual configuration is an antipattern. These are the
  infrastructure needed to build a microservice:

A clear versioning semantics. You can use [Semantic Versionng](http://semver.org) for this. It's
widely used and understood.

A source code control and good branching system. It looks like [Git](https://git-scm.com) and
[Mercurial](https://www.mercurial-scm.org) is dominant nowadays. Create a branching system that
allows you to easily deploy your software. I'm using a [rebase
workflow](http://kensheedlo.com/essays/why-you-should-use-a-rebase-workflow/). There are a lot of
arguments on the workflows. I choose rebasing because it creates a straight tree that easy to be
rolled back and easy to maintain.

A build system. Jenkins is very common to be used on this space. Use it to build the artifacts for
every part of the services end-to-end. Never build anything manually.

A configuration management and service discovery. You should be able to spawn a new instane of a
service and register it presence and read the config right after it's launched without extra human
touch. There are two widely used software: [Zookeeper](https://zookeeper.apache.org) and
[Consul.io](https://consul.io).

An orchestration tool. We want be able to provision and update our clusters automatically. For this,
[Ansible](http://www.ansible.com) is also widely used.

A dashboard. Gather your server performance data. There are many SaaS over there that do this so you
don't need to write it yourself.  [DataDog](https://www.datadoghq.com) and [New
Relic](http://newrelic.com) are widely used for this purpose.

All of above ingredients are essential for successful microservice development and deployment. Those are things
needed to avoid most of the [microservice
anti-pattern](http://www.infoq.com/articles/seven-uservices-antipatterns).

## Conclusion

Microservices are made for a purpose. It enables the business to move more rapidly, new features shipped quickly in
a fine-grained services and team. Building microservices, however, need a disciplined platform. 

I hope this article gives you something to ponder about to evaluate your current architecture. Start small, divide
your system into domains, isolate it, and deploy it independently. Implement unit tests, and integration tests as
early as possible and works towards microservices infrastructure from there.
