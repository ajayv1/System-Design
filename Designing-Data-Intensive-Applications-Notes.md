# Designing Data Intensive Applications (2nd edition) Notes

Although being one of the most important books for the software industry, as it bridges the gap between distributed systems theory and practical engineering, I struggled to find a good summarized reading notes that covers up all the key points of the book, so here it is, I hope.

This reading notes are biased towards *what to do* rather than *how it works*. The main goal behind it is to be a quick one page look-up for people wishing to remember some of details on the fly, or for someone who wish to recap the highlights of the whole book in less than an hour. However, a fair amount of *how it works* explanation details are included.

## Table of Content
- [Part I: Trade-Offs in Data Systems Architecture](#p1)
	- [Chapter 1: Operational Versus Analytical Systems](#ch1)
	- [Chapter 2: Cloud Versus Self-Hosting](#ch2)
	- [Chapter 3: Distributed Versus Single-Node Systems](#ch3)
	- [Chapter 4: Data Systems, Law, and Society](#ch4)
- [Part II: Distributed Data](#p2)
	- [Chapter 5: Replication](#ch5)
	- [Chapter 6: Partitioning](#ch6)
	- [Chapter 7: Transactions](#ch7)
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
