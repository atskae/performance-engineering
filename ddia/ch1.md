# Chapter 1: Trade-Offs in Data Systems Architecture

Different *main* challenges based on the type of application:
* **Data-intensive**: data management: moving/storing huge amounts of data
    * Ex) Social media apps, data analytics, online shopping transactions
* **Compute-intensive**: parallelizing expensive computations
    * CPU/GPU
    * Ex) Weather simulations, graphics/rendering, scientific computing

Applications can be both data-intensive and compute-intensive
* Ex. machine learning training

Standard building blocks in a *data-intensive* application:
* **Databases**: Storing data for other applications to read
* **Caching**: Save results of expensive computations for quick re-use
* **Search indexes**: Search data by keywords
* **Stream processing**: Handle data/events in real-time
* **Batch processing**: Compute metrics of accumulated data periodically

## Operational Versus Analytical Systems

* **Operational**: data infrastructure, backend services that *create* new data based on application users
    * Store/modify data in databases
* **Analytical**: read-only copy of operational data that was transformed to be optimized for data analytics

## Cloud Versus Self-Hosting

An organization needs to decide whether to do:
* In-house - if it is competitive advantage, core of the business
* Outsource - non-core, common/routine
    * Ex) Most companies don't build their own CPUs so they buy them from semiconductor companies

In software specifically, the important questions are:
* Who develops the software
* Who deploys the software

Spectrum of that decision:
| More control, greater investment | <---> | Less control, lower investment |
| - | - | - |
| In-house SW | Off-the-shelf SW | Off-the-shelf SW |
| In-house operations | In-house operations | Outsourced operations |
| Ex. Application code | Ex. Self-hosted database/Iaas | Ex. Cloud services / SaaS |


### Pros and Cons of Cloud Services

Use a cloud service, as opposed to deploying/maintaining a service in-house, if:
* Don't have expertise in deployment
* Unpredictable workloads - the cloud service can reduce and increase resources on demand
    * Ex) provision more resources on expensive computations, then return unused resources. As opposed to having a fixed number of resources and have them sit idle for most of the time
* Training/having a big operations team is expensive - better to have a few to use the cloud service
* Possible that outsourcing operations to a cloud service results in better service, since the cloud service focuses their expertise on that and built experience from having many customers

Cons of using a cloud service:
* No control over it
* Need to wait for a requested feature
* If it goes down, all you can do is wait
* No access to internals (logs, performance metrics) so difficult to debug if something goes wrong
* Vendor-lock-in problem - if the vendor forces you to migrate/upgrade you will have to, which can be expensive
* If cloud provider is in a different country, politics and sanctions can lock you out of your service
* Need to be trusted with data security - not guaranteed and can complicate privacy/security regulations


### Cloud Native System Architecture
**Cloud native**: data systems architecture that take advantage of cloud services

* **Online Transaction Processing (OLTP)**: interactive operations from user input
* **Online Analytical Processing (OLAP)**: doing analysis on the user data obtained from OLTP

| Category | Self-hosted Systems | Cloud native Systems |
| - | - | - |
Operational / OLTP | MySQL, PostgreSQL, MongoDB | AWS Aurora, Azure SQL, DB Hyperscale, Google Cloud Spanner
| Analytical/OLAP | Teradata, ClickHouse, Spark | Snowflake, Google BigQuery, Azure Synapse Analytics |


## Cloud Computing Versus Supercomputing
* Supercomputing/HPC - computationally heavy tasks (ex. scientific computing)
* Cloud computing - online services, need to be available all the time and continually servce users

## Data Systems, Law, and Society
* Data systems have technical requirements but also need to think about legal/societal/ethical requirements
    * The needs of the people it is serving
* **General Data Protection Regulation** (GDPR) - data protection laws regarding how personal data of EU citizens can be stored/used
    * The right for a user to have their data erased: "right to be forgotten"
    * How to deal with immutable data? (ex. logs)
* Not only costs to store/maintain data, but also think about the cost if the data to be leaked, or legal trouble if the systemw was not compliant to the law
* *Data minimization*: only store data that is required
    * As opposed to the "big data philosophy": keep as much data *in-case* it is useful later
* **Service Organization Control (SOC) Type 2** standards
