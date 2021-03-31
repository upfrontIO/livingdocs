---
title: High availbility setup
menu: operations
---

_Architecture diagrams master files can be found [on google drive](https://docs.google.com/document/d/1TQhW3HtzurI78kinxUCqf5OxG3RSoiEURXEn4AIztP0/edit#heading=h.snjxmn1ywzn)_

### Editing stack
The editing stack is usually accessed by a limited number of editorial users. For most setups, the requirements for redundancy are lower than in the delivery stack. It does not have a caching layer. 

{{< img src="images/architecture-editing.png" >}}

- **Editor**: It is unlikely that it needs scaling as it serves static assets only. In case it needs to be redundant, you can always run two instances behind a load balancer.
- **Editing servers**: Scaled based on the load generated by the editorial users. Because of the nature of node.js processes, it is recommended to always build in redundancy and run at least three processes, even in a simple production setup.
- **Elasticsearch (ES)**: Scaled based on the number of objects in the database (mainly articles and pages) and read/write throughput. The load generated by the editorial users is unlikely a need to scale. Redundancy can be achieved through [configuring shards and replicas](https://www.elastic.co/guide/en/elasticsearch/guide/current/scale.html).
- **Postgres (PG)**: Scaled based on the number of objects in the database (mainly articles and pages) and read/write throughput. The load generated by the editorial users is unlikely a need to scale. Redundancy can be achieved through standby servers and failover.
- **Redis**: Scaled based on the number of objects in the key value store (mainly routes for articles and pages and queue jobs) and read/write throughput. Redundancy can be achieved through [configuring replication](https://redis.io/topics/replication).

### Delivery stack
{{< img src="images/architecture-delivery.png" >}}

- **Caching**: The entry point is a Varnish cache server. Redundancy can be achieved through DNS load balancing.
- **Delivery applications**: Should be scaled based on the load generated by the end users. Because of the nature of node.js processes, it is recommended to always build in redundancy and run at least three processes, even in a simple production setup.
- **Delivery servers**: Should be scaled based on the load generated by the end users. Because of the nature of node.js processes, it is recommended to always build in redundancy and run at least three processes, even in a simple production setup.
- **Postgres (PG)**: Scaled based on the load generated by the end users. In most cases, it can be a shared database with the editing stack. Separation of the two stacks can be achieved through running a read only slave database within the delivery stack, following the data from the read/write master in the editing stack. Redundancy can be achieved through standby servers and failover.
- **Redis**: Scaled based on the read/write load. In most cases, it can be a shared database with the editing stack. Redundancy can be achieved through [configuring replication](https://redis.io/topics/replication).