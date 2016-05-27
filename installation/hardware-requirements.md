# Hardware requirements

The hardware requirements greatly depend on your use case. However, you can find a general evaluation and some example installations below. 

## General assessment

We recommend to use the provided Docker containers, so there might be additional overhead for managing the images. 

### Application

For high availability, you can put a load balancer in front of multiple instances of both the server and editor.

#### Editor

Serving static files is a lightweight task. A basic Amazon S3 instance or a simple nginx installation should be sufficient. CPU, memory and disk requirements are low. Even with a large number of people working in the editor, you should not hit limitations quickly. 

#### Server

The servers CPU requirements compares to the ones of a regular node application. One process requires about 250MB memory. Disk is not an issue since the application footprint is low and uploaded assets are stored in the cloud. You can expect to hit limitations earlier in the server than in the editor. This of depends on the number of users and the tasks they and developers perform. 

### Databases

The requirements for the databases of course depend on the amount of data you want to store. Also on the interactions with the database through users and developers. Depending on your requirements, you might want to think about replication. 

#### Postgresql

The database requirements are comparable to any other postgres installation. There is an [extensive guide on postgres hardware](https://wiki.postgresql.org/wiki/Database_Hardware) optimized for high performance.   

#### Elasticsearch

Elasticsearch is more memory hungry than the rest of the stack. It should mostly be light on CPU though. Elasticsearch provides a good guide on [the hardware requirements](https://www.elastic.co/guide/en/elasticsearch/guide/master/hardware.html). 


### Services

#### Cloud storage

The storage your installation needs is directly coupled to the documents you upload and store. Cloud storage is easily scalable, so you should not expect any troubles here.


## Real life examples

Below you can find an overview of real life installations. 


### Simple dokku cloud

We are running a private Dokku cloud with multiple Docker based installations on virtual servers. There are about 10 demo instances running with each a server, editor, postgres,   

Specs | |
:--- | ---
Instance | 2x Flex-8 from Cloudscale
Instances | 10 x Elasticsearch, Postgres, Server, Editor
vCpu | 4
Memory | 8 GB


### NZZ

The production setup at NZZ is hosted on Amazon cloud services and managed by a third party service provider.
 
- 100+ journalists working in the editor
- ~100k documents in the database 
- Planning to import 1.6m documents. The import itself is expected to be heavy on the servers, but no massive scaling required for the daily operations
- An external delivery system with Varnish is used to deliver articles to the user

Service | Specs | |
:--- | :--- | ---
**Editor** | Amazon S3 with local cloudfront CDN
**Server** | 2x Amazon EC2 
| | Instance |  M3 xlarge instances with a load balancer
| | vCPU | 4 
| | Memory | 15 GB 
| | SSD | 40 GB
**Postgres** | 1 x M3 xlarge Amazon RDS
| | vCPU | 4 
| | Memory | 15 GB
| | SSD | 80 GB
**Elasticsearch** | Hosted instance at elastic.co 
| | Cluster size | 2 (1 replica)
| | Memory | 4 GB
| | SSD | 64 GB
**Storage** | Amazon S3