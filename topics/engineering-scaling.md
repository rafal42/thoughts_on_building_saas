<div style="text-align: justify">

# What you'll find here

This is a (far from exhaustive) list of patterns that can be used to scale a system.

# Relational Databases

Looks like this is the biggest part of this page so far. Enjoy.

## Indexes

Indexes are a double-edged sword.

- They _usually_ provide faster data access for _read_ traffic, reducing resource utilization
- Every index increases time necessary for _write_ traffic. The query optimizer might - under some circumstances, depending on the DB Engine - choose a suboptimal index. Some engines offer "query hints" / "index hints" to force the database to use a specific index for given query

The two index types supported by most engines are:
- binary tree - great for range search or string prefix search
- hash - if you only need exact-match queries

When constructing a composite (made of a few different columns) b-tree index, it is important to remember that the order of columns specified in the index matters - it follows the binary tree structure.

Postgres also offers partial indexes (using the `ADD INDEX ... WHERE $x` clause).


## Foreign Key Constraints

Most resources on the Internet I've read may be summed as: _Always add foreign key constraints_. I do not think I'd advise for _always_.

It is true that FK constraints are great for ensuring that if there is a dataset spread over a few tables, and that dataset "makes sense" (or, carries valuable information) only if entries in those few tables exist all at once or none at all. There certainly are situations where database tables are modeled like that. But you should first assess if this is your case.

You should do so, because foreign key constraints bear costs. There is a tiny overhead for when you write data, but it is usually negligible. What is way worse is a potential for surprise-surprise deadlocks. I have seen it come up at two large codebases with different database engines.

As the codebase had grown, there was a set of "core resources" in the system. There were several call sites over the whole system which made a `SELECT FOR UPDATE` to the core resource. If a "child resource" had a foreign key constraint on the `core_resource_id`, any `UPDATE SET ...` on the child resource concurrent  to the `SELECT FOR UPDATE` statement would result in a table lock.

## Data archival or deletion

The strategies you may employ depend on any local laws/regulations, as well as the contracts you have with your clients as to keeping their data. But one thing is certain: you should be able to either delete or archive a portion of data.

Over the lifetime of a system, it is likely that the majority of accounts registered in your system will be orphaned - not deleted, but likely also not actively used. Why would you want to allow all the old data to sit alongside the rest of your transactional data, impacting the performance of the system for active accounts?

Deletion is an easy action, though irreversible. I'd advise to drop data when you're certain there is no value in them (eg. you have other methods to gain similar / better understanding about user behaviour as in the stale data).

Archival is often harder. When you want archival, you may just be playing the "soft-delete" game (the easier version), or you actually may need to sometimes re-ingest the archived data into your transactional data stores.

At the end of the day, either of those strategies should relieve your infrastructure from a significant data volume.

## Partitioning

I'll base this section on the partitioning available in Postgres, as that's what I have experience with. I had the pleasure to move out a poorly performing table with 1.6 billion of out of an old MySQL installation to Postgres, reducing p99 `SELECT` time from 200ms to 3s.

There really isn't a _one proper way to do partitioning_. It all depends on what you want to achieve.
- If you want to have a similar performance for all queries against a partitioned table, you will aim for similarly-sized partitions.
- If you want to optimize the performance of a percentage of queries (think "index on steroids"), you'll likely target the problematic data sets.

There are two types of partitioning:
- Range-based
  - This is "the more flexible" solution; I'd advise to default to it, unless you have no good column to choose for partitioning
  - Allows you to specify a (mostly) arbitrary condition, which a row must meet in order to land in a certain partition
  - The most common scenario for this is time-based partitioning. You can set your partitioning strategy in a way that you will create a single table partition per month.
  - You'll need to ensure that new partitions are added before data is inserted for a given month - otherwise, there will be no table to insert to.
  - You can change the specific ranges for new partitions as they come - this is great for re-scaling. You can start with a partition per quarter; if you start getting more data, you create a partition per month, etc.
- Hash-based
  - This is the solution you'd use if there is no optimal partitioning column for your use case. For example, if you want your data distribution to be (with a small standard deviation) the same across all partitions, you could use  hash partitioning with an auto-increment integer primary key.
  - When it comes to "how does it work", I'll quote Postgres docs here:
    > The table is partitioned by specifying a modulus and a remainder for each partition. Each partition will hold the rows for which the hash value of the partition key divided by the specified modulus will produce the specified remainder.

For any partitioning algorithm, it's important to remember that for queries to actually gain performance boost instead of getting worse, all queries need to include the partition key.

## Sharding

One can think of sharding as "partitioning on steroids". While partitions are separate physical storages within a single database, sharding is a step further - it is storing data in multiple databases.

One can do sharding either for a subset of data which originally resided in a single data store, or for the whole data set. It's easiest to perform for systems which have a full tenant isolation. A tenant-centered design only makes it necessary to keep a single tenant's data within a single shard for fast access. No need to think about how that tenant interacts with other tenants.

Sharding on a database level often goes hand-in-hand with other components of the system. For example, if we're thinking about a single-tenant system, we may end up with an architecture as follows:

- A global sign-in service
- A HTTP Gateway, which routes requests based on the shard the signed-in tenant falls to
- A fleet of HTTP Servers, background/scheduled job processors per tenant

This set-up allows for a complete isolation of shards. One can roll out changes on a per-shard basis.

An alternative would be to only shard the database, and keep all the HTTP Servers / background job processors in a single cluster. One would then need to ensure that all HTTP requests and Background jobs have access to additional metadata, to help with routing queries to an appropriate database.

## Read replicas / follower databases

Most relational databases offer a single-writer setup with multiple followers, which can asynchronously replicate the binlog (MySQL) / write-ahead log (Postgres). Usually, there is more reads than writes in a system.

Moving read throughput to the follower instances provides great relief for the writer, allowing it to concentrate on data ingest. It's especially helpful to move any OLAP-type queries to the followers.

It's important to keep in mind that the replication to followers is asynchronous - we're talking eventual consistency here. For all user experiences to stay intact, it is important that there are no situations where a READ occurs immediately after a WRITE. An example would be this sequence of actions, with no delay between them:
```
1. POST /things -d {"thing_name": "I am a thing"}
2. GET /things
=> {} # Empty object returned, expected a thing there!
```

User Interfaces relying on the API should stray from such implementations. Instead, it is worth to either use data returned from the write-oriented endpoint, or manipulate the state on the client side directly. It's also worthwhile to implement practices such as [optimistic UI](https://www.apollographql.com/docs/react/v2/performance/optimistic-ui/).

# Caching

Usually the cheapest way to scale. The highest return on investment can be achieved when:
- The time to fetch from memory is orders of magnitude smaller than performing an operation to retrieve the data live
- The cache hit rate is high; example situations include:
    - multiple users can see the same OLAP report result
    - a data fetch would otherwise be repeated on every other HTTP request (eg. feature flags state for user)
- The user experience is not hurt by serving stale data

# Configuration as code

This pays dividends according to the tool which you decide to use (or not use) it for.

It's most common to see infrastructure as code - resources across the major cloud providers being provisioned according to specifications in code. Why is this the area that's first implemented? Likely because you end up provisioning a lot of resources, and as your cloud setup grows, maintaining it otherwise would be highly inefficient.

The biggest superpower one gets from configuration as code is the ability to build custom modules. It allows for standardization within an organization and speeds up implementation. It may also be helpful with providing a wider self-service by democratizing access to change the state within a tool, irrespective of RBAC defined for user interfaces.

I'm sure this pays huge dividends in many areas, but I've had success specifically with tools related to monitoring, such as Sentry and Datadog. The ability to spin up a new alert which follows existing guidelines in a few minutes is great. Where it pays off even more, is if you have a "Dashboard template" that 30 of your product teams implement and customize. If you'd want to add another component to that template, you'd have to either:
- Adjust the Terraform module to auto-create a new resource by default
- Manually click through 30 dashboards hoping you make no mistake

I have not used it with a feature flagging tool, but that is also an area where keeping configuration as code would be helpful - especially if you have multiple environments or a large number of experimentation cohorts.
