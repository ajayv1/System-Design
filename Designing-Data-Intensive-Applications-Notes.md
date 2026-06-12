# Designing Data Intensive Applications (2nd edition) Notes

Although being one of the most important books for the software industry, as it bridges the gap between distributed systems theory and practical engineering, I struggled to find a good summarized reading notes that covers up all the key points of the book, so here it is, I hope.

This reading notes are biased towards *what to do* rather than *how it works*. The main goal behind it is to be a quick one page look-up for people wishing to remember some of details on the fly, or for someone who wish to recap the highlights of the whole book in less than an hour. However, a fair amount of *how it works* explanation details are included.

## Table of Content
- [Part I: Trade-Offs in Data Systems Architecture](#p1)
	- [Chapter 1: Operational Versus Analytical Systems](#ch1)
	- [Chapter 2: Cloud Versus Self-Hosting](#ch2)
	- [Chapter 3: Distributed Versus Single-Node Systems](#ch3)
	- [Chapter 4: Data Systems, Law, and Society](#ch4)
- [Part II: Defining Nonfunctional Requirements](#p2)
	- [Chapter 5: Case Study: Social Network Home Timelines](#ch5)
	- [Chapter 6: Describing Performance](#ch6)
	- [Chapter 7: Reliability and Fault Tolerance](#ch7)
	- [Chapter 8: The Trouble with Distributed Systems](#ch8)
	- [Chapter 9: Consistency and Consensus](#ch9)
- [Part III: Derived Data](#p3)
	- [Chapter 10: Batch Processing](#ch10)
	- [Chapter 11: Stream Processing](#ch11)
	- [Chapter 12: The Future of Data Systems](#ch12)
- [More Reading Notes](#more)

## <a name="p1">Part I: Trade-Offs in Data Systems Architecture</a>

We call an application **data-intensive** if data management is one of the primary challenges in developing the application. While in compute-intensive systems the challenge is parallelizing a very large computation, in data-intensive applications we
usually worry more about things like storing and processing large data volumes, managing changes to data, ensuring consistency in the face of failures and concurrency, and making sure services are highly available.

For example, many applications need to do the following:
* Store data so that they, or another application, can find it again later (databases)
* Remember the result of an expensive operation, to speed up reads (caches)
* Allow users to search data by keyword or filter it in various ways (search indexes)
* Handle events and data changes as soon as they occur (stream processing)
* Periodically crunch a large amount of accumulated data (batch processing)

### <a name="ch1">Chapter 1: Operational Versus Analytical Systems</a>

**Operational systems** consist of the backend services and data infrastructure where
data is created—for example, by serving external users. Here, the application
code both reads and modifies the data in its databases, based on the actions
performed by the users.

**Analytical systems** serve the needs of business analysts and data scientists. They
contain a read-only copy of the data from the operational systems, and they are
optimized for the types of data processing that are needed for analytics.

#### Characterizing Transaction Processing and Analytics
An **operational system** typically looks up a small number of records by a key (this is called
a point query). Records are inserted, updated, or deleted based on the user’s input.
Because these applications are interactive, this access pattern became known as online
transaction processing (OLTP).

An **analytical query** scans over
a huge number of records and calculates aggregate statistics (such as count, sum,
or average) rather than returning the individual records to the user. The reports that result from these types of queries are important for BI, helping
management decide what to do next. To differentiate this pattern of using databases
from transaction processing, it has been called online analytical processing (OLAP).

##### Data Warehousing
The data of interest may be spread across multiple operational systems, making
it difficult to combine those datasets in a single query (a problem known as data
silos).

A data warehouse, by contrast, is a separate database that analysts can query to their
hearts’ content, without affecting OLTP operations.

The data warehouse contains a read-only copy of the data from all the various OLTP
systems in the company. Data is extracted from OLTP databases (using either a
periodic data dump or a continuous stream of updates), transformed into an analysisfriendly schema, cleaned up, and then loaded into the data warehouse. This process
of getting data into the data warehouse is known as extract–transform–load (ETL).

<img width="664" height="504" alt="image" src="https://github.com/user-attachments/assets/8db909be-7af5-49ca-a263-6117ef391d26" />

##### Data Lake
Organizations face a need to make data available in a form that is
suitable for use by data scientists. The answer is a data lake: a centralized data
repository that holds a copy of any data that might be useful for analysis, obtained
from operational systems via ETL processes. The difference from a data warehouse is
that a data lake simply contains files, without imposing any particular file format, data model, or schema. Files in a data lake might be collections of database records,
encoded using a file format such as Avro or Parquet (see Chapter 5), but a data
lake can equally well contain text, images, videos, sensor readings, sparse matrices,
feature vectors, genome sequences, or any other kind of data. Besides being
more flexible, a data lake is also often cheaper than relational data storage, since it
can use commoditized file storage such as object stores.

##### Systems of Record and Derived Data
A _system of record_, also known as a source of truth, holds the authoritative or
canonical version of data. When new data comes in—for example, as user input—
it is first written here. Each fact is represented exactly once (the representation
is typically normalized; If there is any discrepancy between another system and the system of record,
the value in the system of record is (by definition) the correct one.

Data in a _derived system_ is the result of taking existing data from another system
and transforming or processing it in some way. If you lose derived data, you
can re-create it from the original source. A classic example is a cache: data can
be served from the cache if present, but if the cache doesn’t contain what you
need, you can fall back to the underlying database. Denormalized values, indexes,
materialized views, transformed data representations, and models trained on a
dataset also fall into this category.

### <a name="ch2">Chapter 2: Cloud Versus Self-Hosting</a>
Ultimately, this is a question about business priorities. A common rule of thumb
is that things that are a core competency or a competitive advantage of your orga‐
nization should be done in-house, whereas things that are non-core, routine, or
commonplace should be left to a vendor.

### <a name="ch3">Chapter 3: Distributed Versus Single-Node Systems</a>
A system that involves several machines communicating via a network is called a
_distributed system_. Each of the processes participating in a distributed system is called
a _node_. You might want to use this type of system for various reasons:

**_Inherent distribution_**
- If an application involves two or more interacting users, each using their own
device, then the system is unavoidably distributed: the communication between
the devices will have to occur via a network.

**_Requests between cloud services_**
- If data is stored in one service but processed in another, that data must be
transferred over the network from one service to the other. Cloud native systems
and microservices are therefore distributed.

**_Fault tolerance/high availability_**
- If your application needs to continue working even if one machine (or several
machines, or the network, or an entire datacenter) goes down, you can use
multiple machines to give you redundancy. When one fails, another one can take
over.

**_Scalability_**
- If your data volume or computing requirements grow bigger than a single
machine can handle, you can potentially spread the load across multiple
machines.

**_Latency_**
- If you have users around the world, you might want to have servers in various
regions worldwide so that each user can be served from a server that is geograph‐
ically close to them. That avoids the users having to wait for network packets
to travel halfway around the world to answer their requests.

**_Elasticity_**
- If your application is busy at some times and idle at others, a cloud deployment
can scale up or down to meet the demand so that you pay only for resources you
are actively using. This is more difficult on a single machine, which needs to be
provisioned to handle the maximum load, even at times when it is barely used.

**_Specialized hardware_**
- Different parts of the system can take advantage of different types of hardware
to match their workload. For example, an object store may use machines with
many disks but few CPUs, whereas a data analysis system may use machines with
lots of CPU and memory but no disks, and a machine learning system may use
machines with GPUs (which are much more efficient than CPUs for training
deep neural networks and other ML tasks).

**_Legal compliance_**
- Some countries have data residency laws that require data about people in their
jurisdiction to be stored and processed geographically within that country.
The scope of these rules varies—for example, in some cases it applies only to
medical or financial data, while other cases are broader. A service with users
in several such jurisdictions will therefore have to distribute their data across
servers in several locations.

**_Sustainability_**
- If you have flexibility on where and when to run your jobs, you might be able
to run them at a time and in a place where plenty of renewable electricity is
available and avoid running them when the power grid is under strain. This can
reduce your carbon emissions and allow you to take advantage of cheap power
when it is available.

These reasons apply to both services that you write yourself (application code) and
services consisting of off-the-shelf software (such as databases).

#### Problems with Distributed Systems
Distributed systems also have downsides. Every request and API call that traverses
the network needs to deal with the possibility of failure. The network may be inter‐
rupted, or the service may be overloaded or crash, and therefore any request may
time out without receiving a response. In this case, we don’t know whether the service
received the request, and simply retrying it might not be safe. We will discuss these
problems in detail in Chapter 9

Although datacenter networks are fast, making a call to another service is still vastly
slower than calling a function in the same process [43]. When operating on large
volumes of data, rather than transferring the data from storage to a separate machine
that processes it, it can be faster to bring the computation to the machine that
already has the data [44]. More nodes are not always faster; in some cases, a simple
single-threaded program on one computer can perform significantly better than a
cluster with over 100 CPU cores [45].

Troubleshooting a distributed system is often difficult—if the system is slow to
respond, how do you figure out where the problem lies? Techniques for diagnosing
problems in distributed systems are developed under the heading of observability [46,
47], which involves collecting data about the execution of a system and allowing that
data to be queried in ways that allow both high-level metrics and individual events
to be analyzed. Tracing tools such as OpenTelemetry, Zipkin, and Jaeger allow you to
track which client called which server for which operation and how long each call
took.

Databases provide various mechanisms for ensuring data consistency, as we shall see
in Chapters 6 and 8. However, when each service has its own database, maintaining
consistency of data across those different services becomes the application’s problem.
Distributed transactions, which we explore in Chapter 8, are a possible technique for
ensuring consistency, but they are rarely used in a microservices context because they
run counter to the goal of making services independent from each other, and many
databases don’t support them [49].

For all these reasons, performing a task on a single machine is often much simpler
and cheaper than setting up a distributed system [22, 45, 50]. CPUs, memory, and
disks have grown larger, faster, and more reliable. When combined with single-node
databases such as DuckDB, SQLite, and KùzuDB, many workloads can now run on a
single node. We will explore this topic further in Chapter 4.

#### Microservices and Serverless
The most common way of distributing a system across multiple machines is to divide
them into clients and servers and let the clients make requests to the servers. Most
commonly, HTTP is used for this communication, as we will discuss in “Dataflow
Through Services: REST and RPC” on page 180. The same process may be both a
server (handling incoming requests) and a client (making outbound requests to other
services).

This way of building applications has traditionally been called a service-oriented
architecture (SOA); more recently, the idea has been refined into a microservices
architecture [51, 52]. In a microservices architecture, a service has one well-defined
purpose (e.g., in the case of S3, this is file storage); each service exposes an API
that can be called by clients via the network, and each service has one team that is
responsible for its maintenance. A complex application can thus be decomposed into
multiple interacting services, each managed by a separate team.

Dividing a complex piece of software into multiple services has several advantages:
each service can be updated independently, reducing coordination effort among
teams; each service can be assigned the hardware resources it needs; and hiding the
implementation details behind an API means the service owners are free to change
the implementation without affecting clients. In terms of data storage, it is common
for each service to have its own databases and not to share databases between
services. Sharing a database would effectively make the entire database structure a
part of the service’s API, and then that structure would be difficult to change. Shared
databases could also cause one service’s queries to negatively impact the performance
of other services.

On the other hand, having many services can itself breed complexity. Testing a ser‐
vice during development can be complicated, since you also need to run all the other
services that it depends on. What’s more, each service requires infrastructure for
deploying new releases, adjusting the allocated hardware resources to match the load,
collecting logs, monitoring service health, and alerting an on-call engineer in the case
of a problem. Orchestration frameworks such as Kubernetes have become a popular
way of deploying services, since they provide a foundation for this infrastructure.

In addition, microservice APIs can be challenging to evolve. Clients that call an
API expect it to have certain fields. Developers might wish to add or remove fields
in an API as business needs change, but doing so can cause clients to fail. Worse
still, such failures are often not discovered until late in the development cycle, when
the updated service API is deployed to a staging or production environment. API
description standards such as OpenAPI and gRPC help manage the relationship
between client and server APIs; we discuss these further in Chapter 5.

Microservices are primarily a technical solution to a people problem: allowing differ‐
ent teams to make progress independently without having to coordinate with each
other. This is valuable in a large company, but in a small company with fewer teams,
using microservices is likely to be unnecessary overhead, and implementing the
application in the simplest way possible is preferable [51].

### <a name="ch4">Chapter 4: Data Systems, Law, and Society</a>
As you’ve seen in this chapter, the architecture of data systems is influenced not only
by technical goals and requirements, but also by the human needs of the organiza‐
tions that they support. Increasingly, data systems engineers are realizing that serving
the needs of their own business is not enough; we also have a responsibility toward
society at large.

One particular concern is systems that store data about people and their behavior.
Since 2018, the **GDPR** has given residents of many European countries greater con‐
trol and legal rights over their personal data, and similar privacy regulations have
been adopted in various other countries and states around the world (including, for
example, the **CCPA**). Regulations around AI, such as the EU AI Act, place further
restrictions on how personal data can be used.

Moreover, even in areas that are not directly subject to regulation, there is increas‐
ing recognition of the effects that computer systems have on people and society.
Social media has changed how individuals consume news, which influences their
political opinions and hence may affect the outcome of elections. Automated systems
increasingly make decisions that have profound consequences for individuals, such
as who should be given a loan or insurance coverage, who should be invited to a job
interview, or who should be suspected of a crime [59].

Everyone who works on such systems shares a responsibility for considering the
ethical impact of their decisions and ensuring that they comply with relevant laws.
Not everyone needs to become an expert in law and ethics, but a basic awareness of
legal and ethical principles is just as important as, say, some foundational knowledge
in distributed systems.

Legal considerations are influencing the very foundations of data system design [60].
For example, the GDPR grants individuals the right to have their data erased on
request (sometimes known as the right to be forgotten). However, as we shall see in
this book, many data systems rely on immutable constructs such as append-only logs
as part of their design. How can we ensure deletion of some data in the middle of a
file that is supposed to be immutable? How do we handle deletion of data that has
been incorporated into derived datasets (see “Systems of Record and Derived Data”
on page 10), such as training data for ML models? Answering these questions creates
new engineering challenges.

In general, we store data because we think that its value is greater than the costs of
storing it. However, it is worth remembering that the costs of storage extend beyond
the bill you pay for S3 or another service. The cost-benefit calculation should also
take into account the risks of liability and reputational damage if the data were to
be leaked or compromised by adversaries, and the risk of legal costs and fines if the
storage and processing of the data is found not to be compliant with the law [50].
Governments or police forces might also compel companies to hand over data.

When data could reveal criminalized behaviors (e.g., homosexuality in several Middle
Eastern and African countries, or seeking an abortion in several states in the United
States), storing that data creates real safety risks for users. Travel to an abortion clinic,
for example, could easily be revealed by location data, or perhaps even by a log of the
user’s IP addresses over time (which indicate approximate location).

Once all the risks are taken into account, it might be reasonable to decide that some
data is simply not worth storing, and that it should therefore be deleted. This princi‐
ple of data minimization (sometimes known by the German term Datensparsamkeit)
runs counter to the “big data” philosophy of storing lots of data speculatively in case
it turns out to be useful in the future [61]. But data minimization fits with the GDPR,
which mandates that personal data may be collected only for a specified, explicit
purpose; cannot later be used for any other purpose; and must not be kept for longer
than necessary for the purposes for which it was collected [62].

Businesses have also taken notice of privacy and safety concerns. Credit card compa‐
nies require payment processing businesses to adhere to strict Payment Card Indus‐
try (PCI) standards. Processors undergo frequent evaluations from independent
auditors to verify continued compliance. Software vendors have also seen increased
scrutiny. Many buyers now require their vendors to comply with Service Organiza‐
tion Control (SOC) Type 2 standards. As with PCI compliance, vendors undergo
third-party audits to verify adherence.

Generally, it is important to balance the needs of your business against the needs of
the people whose data you are collecting and processing. There is much more to this
topic; in Chapter 14 we will go deeper into ethics and legal compliance, including the
problems of bias and discrimination. 

## <a name="p2">Part II: Defining Nonfunctional Requirements</a>
If you are building an application, you will be driven by a list of requirements. At the
top of your list is most likely the functionality that the application must offer: what
screens and what buttons you need, and what each operation is supposed to do in
order to fulfill the purpose of your software. These are your **functional requirements**.

In addition, you probably have **nonfunctional requirements**: for example, the app
should be fast, reliable, secure, legally compliant, and easy to maintain. These require‐
ments might not be explicitly written down, because they may seem somewhat obvi‐
ous, but they are just as important as the app’s functionality; an app that is unbearably
slow or unreliable might as well not exist.

Many nonfunctional requirements, such as security, fall outside the scope of this book.
But we will consider a few, and this chapter will help you articulate them for your own
systems. In particular, we will look at the following:
- Defining and measuring the performance of a system
- What it means for a service to be reliable—namely, continuing to work correctly,
even when things go wrong
- Allowing a system to be scalable by having efficient ways of adding computing
capacity as the load on the system grows
- Making it easier to maintain a system in the long term

### <a name="ch5">Chapter 5: Case Study: Social Network Home Timelines</a>
Imagine we have been given the task of implementing a social network in the style of
X, where users can post messages and follow other users. This will
be a huge simplification of how such a service actually works, but it will help
illustrate some of the issues that arise in large-scale systems.

Let’s assume that users make a total of 500 million posts per day, or 5,800 posts
per second on average. Occasionally, the rate can spike to as high as 150,000 posts
per second [4]. Let’s also assume that the average user follows 200 people and has
200 followers (although there is a very wide range: most people have only a handful
of followers, and a few celebrities, such as Barack Obama, have over 100 million
followers).

#### Representing Users, Posts, and Follows
We keep all the data in a relational database, as shown in Figure 2-1. We have one
table for users, one table for posts, and one table for follow relationships.

<img width="655" height="266" alt="image" src="https://github.com/user-attachments/assets/207d3496-f476-49e1-b607-01a28abaaa84" />

Let’s say the main read operation that our social network must support is the home
timeline, which displays recent posts by people the user is following (for simplicity
we will ignore ads, suggested posts from people they are not following, and other
extensions). We could write the following SQL query to get the home timeline for a
particular user:

<code>SELECT posts.*, users.* FROM posts
 JOIN follows ON posts.sender_id = follows.followee_id
 JOIN users ON posts.sender_id = users.id
 WHERE follows.follower_id = current_user
 ORDER BY posts.timestamp DESC
 LIMIT 100</code>

 To execute this query, the database will use the follows table to find everybody who
current_user is following, look up recent posts by those users, and sort them by
timestamp to get the most recent 1,000 posts by any of the followed users.

Posts are supposed to be timely, so let’s assume that after somebody makes a post, we
want their followers to be able to see it within five seconds. One approach is for the
user’s client to repeat the preceding query every five seconds while the user is online
(this is known as polling). If we assume that 10 million users are online and logged
in at the same time, that would mean running the query 2 million times per second.
Even if we were to poll less frequently, this is a lot.

This query is also quite expensive: if a user is following 200 people, the query needs
to fetch a list of recent posts by each of those 200 people and merge those lists. Two
million timeline queries per second times 200 followed accounts makes 400 million
lookups per second—a huge number. And that’s the average case. Some users follow
tens of thousands of accounts; for them, this query is very expensive to execute and
difficult to make fast.

#### Materializing and Updating Timelines
How can we do better? First, instead of polling, it would be better if the server
actively pushed new posts to any followers who are currently online. Second, we
should precompute the results of the query so that a user’s request for their home
timeline can be served from a cache.

Imagine that for each user, we store a data structure containing their home timeline
(i.e., the recent posts by people they are following). Every time a user makes a
post, we look up all their followers and insert that post into the home timeline of
each follower—like delivering a message to a mailbox. Now when a user logs in,
we can simply give them this precomputed home timeline. Moreover, to receive a
notification about any new posts on their timeline, the user’s client simply needs to
subscribe to the stream of posts being added to their home timeline.

The downside of this approach is that we now need to do more work every time
a user makes a post, because the home timelines are derived data that needs to be
updated. The process is illustrated in Figure 2-2. When one initial request results in
several downstream requests being carried out, we use the term fan-out to describe
the factor by which the number of requests increases.

<img width="666" height="253" alt="image" src="https://github.com/user-attachments/assets/04661c6f-2a5a-4a95-a302-bc60d5ad1160" />

At a rate of 5,800 posts per second, if the average post reaches 200 followers (i.e., a
fan-out factor of 200), we will need to do just over 1 million home timeline writes
per second. This is a lot, but it’s still a significant saving compared to the 400 million
per-sender post lookups per second that we would otherwise have to do.

If the rate of posts spikes because of a special event, we don’t have to do the timeline
deliveries immediately—we can enqueue them and accept that it will temporarily take
a bit longer for posts to show up in followers’ timelines. Even during such load spikes,
timelines remain fast to load, since we simply serve them from a cache.

This process of precomputing and updating the results of a query is called materialization, and the timeline cache is an example of a materialized view (a concept we will
discuss further in later chapters). The materialized view speeds up reads, but in return
we have to do more work on writes. The cost of writes for most users is modest, but a
social network also has to consider some extreme cases:

- If a user is following a very large number of accounts, and those accounts post
a lot, that user will have a high rate of writes to their materialized timeline.
However, that user is not likely reading all the posts in their timeline, so it’s OK to
simply drop some of their timeline writes and show the user only a sample of the
posts from the accounts they’re following [5].

- When a celebrity account with a very large number of followers makes a post,
we have to do a lot of work to insert that post into the home timelines of each
of their millions of followers. In this case, dropping some of those writes is
not OK. One way of solving this problem is to handle celebrity posts separately
from everyone else’s posts: we can save ourselves the effort of adding celebrity
posts to millions of timelines by storing them separately and merging them with
the materialized timeline when it is read. Despite such optimizations, handling
celebrities on a social network can require a lot of infrastructure [6].

### <a name="ch6">Chapter 6: Describing Performance</a>
Most discussions of software performance consider two main types of metric:

_Response time_
The elapsed time from the moment when a user makes a request until they
receive the requested answer. The unit of measurement is seconds (or milli‐
seconds, or microseconds).

_Throughput_
The number of requests per second, or the data volume per second, that the
system is processing. For a given allocation of hardware resources, there is a
maximum throughput that can be handled. The unit of measurement is “somethings per second.”

In the social network case study, “posts per second” and “timeline writes per second”
are throughput metrics, whereas “time it takes to load the home timeline” and “time
until a post is delivered to followers” are response time metrics.

Throughput and response time are often related. An example of such a relationship
for an online service is sketched in Figure 2-3. The service has a low response
time when request throughput is low, but response time increases as load increases.
This is because of queueing: when a request arrives on a highly loaded system, the
CPU is likely already in the process of handling an earlier request, and therefore
the incoming request needs to wait until the earlier request has been completed. As
throughput approaches the maximum that the hardware can handle, queueing delays
increase sharply.

<img width="659" height="325" alt="image" src="https://github.com/user-attachments/assets/b466f0fa-3345-492b-a6d2-c4661060ebc3" />

#### When an Overloaded System Won’t Recover
If a system is close to overload, with throughput pushed close to the limit, it can
sometimes enter a vicious cycle where it becomes less efficient and hence even
more overloaded. For example, if a long queue of requests is waiting to be handled,
response times may increase so much that clients time out and resend their requests.
This causes the rate of requests to increase even further, making the problem worse—
a retry storm. Even when the load is reduced again, such a system may remain in an
overloaded state until it is rebooted or otherwise reset. This phenomenon is called a
metastable failure, and it can cause serious outages in production systems [7, 8, 9].

To avoid retries overloading a service, you can increase and randomize the time
between successive retries on the client side (exponential backoff [10, 11]) and tem‐
porarily stop sending requests to a service that has returned errors or timed out
recently (by using a circuit breaker [12, 13] or token bucket algorithm [14]). The
server can also detect when it is approaching overload and start proactively rejecting
requests (load shedding [15]), or send back responses asking clients to slow down
(backpressure [1, 16]). The choice of queueing and load balancing algorithms can also
make a difference [17].

In terms of performance metrics, the response time is usually what users care about
the most, whereas the throughput determines the required computing resources (e.g.,
how many servers you need) and hence the cost of serving a particular workload. If
throughput is likely to increase beyond the current hardware’s capability, the capacity
needs to be expanded; a system is said to be scalable if its maximum throughput can
be significantly increased by adding computing resources.

#### Latency and Response Time
“Latency” and “response time” are sometimes used interchangeably, but in this book
we will use these and a few related terms in a specific way (illustrated in Figure 2-4):

- The response time is what the client sees; it includes all delays incurred anywhere
in the system.

- The service time is the duration for which the service is actively processing the
client’s request.

- Queueing delays can occur at several points in the flow—for example, after a
request is received, it might need to wait until a CPU is available before it can be
processed, or a response packet might need to be buffered before it is sent over
the network if other tasks on the same machine are sending a lot of data via the
outbound network interface.

- Latency is a catchall term for time during which a request is not being actively
processed—that is, during which it is latent. In particular, network latency or
network delay refers to the time that a request and response spend traveling
through the network.

<img width="661" height="283" alt="image" src="https://github.com/user-attachments/assets/c52adcb2-4158-4222-9fe9-38c5d03e4c1d" />

The response time can vary significantly from one request to the next, even if you
keep making the same request over and over again. Many factors can add random
delays—for example, a context switch to a background process, the loss of a network
packet and TCP retransmission, a garbage collection pause, a page fault forcing a read
from disk, or mechanical vibrations in the server rack [18]. We will discuss this topic
in more detail in “Timeouts and Unbounded Delays” on page 352.

Queueing delays often account for a large part of the variability in response times. As
a server can process only a small number of things in parallel (limited, for example,
by its number of CPU cores), it takes only a small number of slow requests to hold
up the processing of subsequent requests—an effect known as head-of-line blocking.
Even if those subsequent requests have fast service times, the client will see a slow
overall response time due to the time waiting for the prior request to complete. The
queueing delay is not part of the service time, and for this reason it is important to
measure response times on the client side.

#### Average, Median, and Percentiles
Because the response time varies from one request to the next, we need to think of
it not as a single number, but as a distribution of values that we can measure. In
Figure 2-5, each gray bar represents a request to a service, and its height shows how
long that request took. Most requests are reasonably fast, but occasional outliers take
much longer. Variation in network delay is also known as **jitter**.

<img width="664" height="259" alt="image" src="https://github.com/user-attachments/assets/9c0499d4-8ca0-45e2-b90c-b65b5860b1f6" />

It’s common to report the average response time of a service (technically, the arith‐
metic mean, which you find by summing all the response times and dividing by the
number of requests). The mean response time is useful for estimating throughput
limits [19]. However, the mean is not a very good metric if you want to know
your “typical” response time, because it doesn’t tell you how many users actually
experienced that delay.

Usually **it’s better to use percentiles**. If you take your list of response times and sort
it from fastest to slowest, the median is the halfway point—for example, if your
median response time is 200 ms, that means half your requests return in less than
200 milliseconds (ms), and half your requests take longer. This makes the median a
good metric if you want to know how long users typically have to wait. The median is
also known as the 50th percentile, sometimes abbreviated as p50.

To figure out how bad your outliers are, you can look at higher percentiles: the
95th, 99th, and 99.9th percentiles are common (abbreviated p95, p99, and p999). For
example, if the 95th percentile response time is 1.5 seconds, that means 95 out of
100 requests take less than 1.5 seconds, and 5 out of 100 requests take 1.5 seconds or
more. This is illustrated in Figure 2-5.

High response-time percentiles, also known as** tail latencies**, are important because
they directly affect users’ experience of the service. For example, Amazon describes
response time requirements for internal services in terms of the 99.9th percentile,
even though this affects only 1 in 1,000 requests. This is because the customers with
the slowest requests are often those who have the most data on their accounts, as they have made many purchases—that is, they’re the most valuable customers [20].

It’s important to keep those customers happy by ensuring the website is fast for them.
Optimizing the 99.99th percentile (the slowest 1 in 10,000 requests) was deemed too
expensive and found to not yield enough benefit for Amazon’s purposes. Reducing
response times at very high percentiles is difficult because they are easily affected by
random events outside of your control, and the benefits are diminishing.

#### Use of Response Time Metrics
High percentiles are especially important in backend services that are called multiple
times as part of serving a single end-user request. Even if you make the calls in
parallel, the request still needs to wait for the slowest of the parallel calls to complete.
It takes just one slow call to make the entire end-user request slow, as illustrated in
Figure 2-6. Even if only a small percentage of backend calls are slow, the chance of
getting a slow call increases if an end-user request requires multiple backend calls, so
a higher proportion of such end-user requests end up being slow (an effect known as
tail latency amplification [27]).

### <a name="ch7">Chapter 7: Reliability and Fault Tolerance</a>

#### Reliability
System should continue to work correctly, even in the face of faults and human errors.

_Fault_
A fault occurs when a particular part of a system stops working correctly—for
example, if a single hard drive malfunctions, or a single machine crashes, or an
external service (that the system depends on) has an outage.

_Failure_
A failure occurs when the system as a whole stops providing the required service
to the user—in other words, when it does not meet the SLO.

The distinction between faults and failures can be confusing because they are the
same thing, just at different levels. For example, if a hard drive stops working, we say
that the hard drive has failed; if the system consists of only that one hard drive, it
has stopped providing the required service and thus has also failed. However, if the
system consists of multiple hard drives, the failure of a single hard drive is only a fault
from the point of view of the bigger system, and the bigger system might be able to
tolerate that fault by having a copy of the data on another hard drive.

#### Fault Tolerance
We call a system fault-tolerant if it continues providing the required service to users
in spite of certain faults occurring. If a system cannot tolerate a certain part becoming
faulty, we call that part a single point of failure (SPOF), because a fault in that part
escalates to cause the failure of the whole system.

Counterintuitively, in such fault-tolerant systems, it can make sense to increase the rate
of faults by triggering them deliberately—for example, by randomly killing individual
processes without warning. This is called fault injection. Many critical bugs are actually
due to poor error handling [40]; by deliberately inducing faults, you ensure that the
fault-tolerance machinery is continually exercised and tested, which can increase your
confidence that faults will be handled correctly when they occur naturally. Chaos engi‐
neering is a discipline that aims to improve confidence in fault-tolerance mechanisms
through experiments such as deliberately injecting faults [41].

Although we generally prefer tolerating faults over preventing faults, in some cases
where prevention is better than cure (e.g., because no cure exists). This is the case
with security matters, for example; if an attacker has compromised a system and
gained access to sensitive data, that event cannot be undone.

Hardware redundancy is the first line of defense against hardware faults. It was sufficient for a long time, but as computing demand increase, there is a move toward systems that can tolerate the loss of entire machines by using software fault tolerance as well.

The reason behind software faults is making some kind of assumptions about the environment, this assumptions are usually true, until the moment they are not. There is no quick solution to the problem, but the software can constantly check itself while running for discrepancy.

Some approaches for making reliable systems, in spite of unreliable human actions include:
- Design abstractions that are minimal and easy to achieve one thing with, but not too restrictive for people to work around them.
- Provide fully featured sandbox environments with real data for testing, without affecting real users.
- Test throughly at all levels, from unit tests, to whole system integration tests.
- Make it fast to roll back configuration changes, and provide tools to re-compute data.
- Use proper monitoring that shows early warnings signals of faults.

#### Scalability
As system grows, there should be reasonable ways for dealing with that growth.

The first step in scaling a system is to define the system's loads parameters (eg. requests, read to write ratio, etc.)

Throughput is the most important metric in batch processing systems, while response time is the most important metrics for online systems.

A common performance metric is percentile, where Xth percentile = Y ms means that X% of the requests will perform better than Y ms. It's important to optimize for a high percentile, as customers with slowest requests often have the most data (eg. purchases). However, over optimizing (eg. 99.999th) might be too expensive.

It's important to measure response times on client side against realistic traffic size.

Elastic system is useful if load is highly unpredictable, but manually scaled systems are simpler and have fewer operational surprises.

In an early stage startup, it's more important to be able to iterate quickly on product features than to scale to some hypothetical future load.

#### Maintainability
Different people who works on the system should all be able to work on it productively

The majority of the cost of the software is in the ongoing maintenance and not the initial development.

A good system should be operable, which means making routine tasks easy. This can be done by:
- Good monitoring
- Avoiding dependency on individual machines
- Good documentation
- Providing good default behavior, while giving administrators the option to override
- Self-healing, while giving administrators a manual control

A good system should be simple, this can be done by reducing complexity, which doesn't necessarily mean reducing its functionality, but rather by making abstractions.

Simple and easy to understand systems are usually easier to modify than complex ones.

A good system should be evolvable, which means making it easily adapt the changes. Agile is one of the best working patterns for maintaining evolvable systems.
