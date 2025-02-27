---
title: Follower Reads Topology
summary: Guidance on using the follower reads topology in a multi-region deployment.
toc: true
docs_area: deploy
---

In a multi-region deployment, [follower reads](follower-reads.html) are a good choice for tables with the following requirements:

- Read latency must be low, but write latency can be higher.
- Reads can be historical.
- Rows in the table, and all latency-sensitive queries, **cannot** be tied to specific geographies (e.g., a reference table).
- Table data must remain available during a region failure.

{{site.data.alerts.callout_success}}
If reads from a table must be exactly up-to-date, use [global tables](global-tables.html) or [regional tables](regional-tables.html) instead. Up-to-date reads are required by tables referenced by [foreign keys](foreign-key.html), for example.
{{site.data.alerts.end}}

## Prerequisites

### Fundamentals

{% include {{ page.version.version }}/topology-patterns/fundamentals.md %}

### Cluster setup

{% include {{ page.version.version }}/topology-patterns/multi-region-cluster-setup.md %}

## Configuration

{{site.data.alerts.callout_info}}
Follower reads requires an [Enterprise license](https://www.cockroachlabs.com/get-cockroachdb).
{{site.data.alerts.end}}

### Summary

You configure your application to use [follower reads](follower-reads.html) by adding an [`AS OF SYSTEM TIME`](as-of-system-time.html) clause when reading from the table. This tells CockroachDB to read slightly historical data from the closest replica so as to avoid being routed to the leaseholder, which may be in an entirely different region. Writes, however, will still leave the region to get consensus for the table.

### Steps

<img src="{{ 'images/v21.2/topology-patterns/topology_follower_reads1.png' | relative_url }}" alt="Follower reads topology" style="max-width:100%" />

Assuming you have a [cluster deployed across three regions](#cluster-setup) and a table like the following:

{% include copy-clipboard.html %}
~~~ sql
> CREATE TABLE postal_codes (
    id INT PRIMARY KEY,
    code STRING
);
~~~

Insert some data:

{% include copy-clipboard.html %}
~~~ sql
> INSERT INTO postal_codes (ID, code) VALUES (1, '10001'), (2, '10002'), (3, '10003'), (4,'60601'), (5,'60602'), (6,'60603'), (7,'90001'), (8,'90002'), (9,'90003');
~~~

1. If you do not already have one, [request a trial Enterprise license](https://www.cockroachlabs.com/get-cockroachdb).

2. <span class="version-tag">New in v21.2:</span> Decide which type of follower read you would like to perform: _exact staleness_ reads, or _bounded staleness_ reads. For more information about when to use each type of read, see [when to use exact staleness reads](follower-reads.html#when-to-use-exact-staleness-reads) and [when to use bounded staleness reads](follower-reads.html#when-to-use-bounded-staleness-reads).

   - To use [_exact staleness_ follower reads](follower-reads.html#exact-staleness-reads), configure your app to use [`AS OF SYSTEM TIME`](as-of-system-time.html) with the [`follower_read_timestamp()` function](functions-and-operators.html) whenever reading from the table:

    {% include copy-clipboard.html %}
    ~~~ sql
    > SELECT code FROM postal_codes
        AS OF SYSTEM TIME follower_read_timestamp()
                WHERE id = 5;
    ~~~

    You can also set the `AS OF SYSTEM TIME` value for all operations in a read-only transaction:

    {% include copy-clipboard.html %}
    ~~~ sql
    > BEGIN;

    SET TRANSACTION AS OF SYSTEM TIME follower_read_timestamp();

      SELECT code FROM postal_codes
        WHERE id = 5;

      SELECT code FROM postal_codes
        WHERE id = 6;

    COMMIT;
    ~~~

    {{site.data.alerts.callout_success}}
    Using the [`SET TRANSACTION`](set-transaction.html#use-the-as-of-system-time-option) statement as shown in the example above will make it easier to use exact staleness follower reads from [drivers and ORMs](install-client-drivers.html).
    {{site.data.alerts.end}}

   - <span class="version-tag">New in v21.2:</span> To use [_bounded staleness_ follower reads](follower-reads.html#bounded-staleness-reads), configure your app to use [`AS OF SYSTEM TIME`](as-of-system-time.html) with the [`with_min_timestamp()` or `with_max_staleness()` functions](functions-and-operators.html) whenever reading from the table. Note that only single-row point reads in single-statement (implicit) transactions are supported.

    {% include_cached copy-clipboard.html %}
    ~~~ sql
    SELECT code FROM postal_codes AS OF SYSTEM TIME with_max_staleness('10s') where id = 5;
    ~~~

## Characteristics

### Latency

#### Reads

Reads retrieve historical data from the closest replica and, therefore, never leave the region. This makes read latency very low but slightly stale.

For example, in the animation below:

1. The read request in `us-central` reaches the regional load balancer.
2. The load balancer routes the request to a gateway node.
3. The gateway node routes the request to the closest replica for the table. In this case, the replica is *not* the leaseholder.
4. The replica retrieves the results as of your preferred staleness interval in the past and returns to the gateway node.
5. The gateway node returns the results to the client.

<img src="{{ 'images/v21.2/topology-patterns/topology_follower_reads_reads.png' | relative_url }}" alt="Follower reads topology" style="max-width:100%" />

#### Writes

The replicas for the table are spread across all 3 regions, so writes involve multiple network hops across regions to achieve consensus. This increases write latency significantly.

For example, in the animation below:

1. The write request in `us-central` reaches the regional load balancer.
2. The load balancer routes the request to a gateway node.
3. The gateway node routes the request to the leaseholder replica for the table in `us-east`.
4. Once the leaseholder has appended the write to its Raft log, it notifies its follower replicas.
5. As soon as one follower has appended the write to its Raft log (and thus a majority of replicas agree based on identical Raft logs), it notifies the leaseholder and the write is committed on the agreeing replicas.
6. The leaseholder then returns acknowledgement of the commit to the gateway node.
7. The gateway node returns the acknowledgement to the client.

<img src="{{ 'images/v21.2/topology-patterns/topology_follower_reads_writes.gif' | relative_url }}" alt="Follower reads topology" style="max-width:100%" />

### Resiliency

Because this pattern balances the replicas for the table across regions, one entire region can fail without interrupting access to the table:

<img src="{{ 'images/v21.2/topology-patterns/topology_follower_reads_resiliency.png' | relative_url }}" alt="Follower reads topology" style="max-width:100%" />

## See also

{% include {{ page.version.version }}/topology-patterns/see-also.md %}
