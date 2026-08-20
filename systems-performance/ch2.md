# Chapter 2: Methodologies

## 2.1 Terminology
* **IOPS**: input operations per second
    * Rate of data transfer
    * Disk I/O: read/writes per second
* **Throughput**: the rate of work performed
    * Communications: *data rate* (bytes/sec, bits/sec)
    * Databases: *operation rate* (operations/transactions per second)
* IOPS vs. throughput
    * IOPS is the number of operations, while throughput is how much data is transferred per second
* **Response time**: total time to finish an operation
    * Includes wait times, transfer times, and actually being serviced
* **Latency**: the time spent waiting to be serviced
* **Utilization**: how busy a resource is, how much time spent on actually doing work
    * *Memory utilizaton*: how much memory was consumed (ex. storage resources)
* **Saturation**: resource has queued work that it cannot service
* **Bottleneck**: anything that limits the performance of a system
    * Goal is to remove bottlenecks if possible
* **Workload**: input/load that is applied to the system
    * Databases: queries/commands submitted by the user
* **Cache**: faster/smaller storage to access, to avoid accessing slower storage

## 2.2 Models
* **System under test** (SUT)
```
                      ---------------------
Input (workload) ---> | System under test | ---> Resulting performance
                      ---------------------
                                ^
                                |
                                |
                           Pertubations
```
* **Pertubations**: interference that affect performance (other users on the system, scheduling activity, network components like load balancers, proxy servers)

* **Queueing systems**
    * **Response time**
```
Arrival -----> | Queue | -----> ---------
                               |         |
                               | Service |
                               |         |
Departure <-----------------   -----------
```

## 2.3 Concepts
### Latency
* **Latency**: the time spent waiting for an operation to be performed
    * Needs to be clearly defined based on context (ex. when loading a webpage, what is defined as *latency*?)

Example:
* Operation: transfer data over the network
* Need to connect first, which introduces latency
* Response time = connection latency + data transfer time
```
Network service request --> | Connection latency        | Data transfer time |
```

### Time Scales
* **Time scales**: scaling how long an operation takes in comparison to other operations to understand the durations better
    * Ex. 3.5 GHz processor with a CPU cycle time of 0.3ns
        * L1 cache access: 0.9ns
        * Internet: San Francisco to NY: 40ms
        * Physical system reboot: 5min
    * Hard to intuitively *feel* the time durations since they are too short for a human to experience the differences, but we can *scale* them to get a clearer picture:
        * Let's say 1 CPU cycle takes 1 second. Then:
            * L1 cache access: 3s
            * Internet: San Francisc to NY: 4 years
            * Physical system reboot: 32 millenia
        * See page 26 for a table of more examples, very fascinating!

### Trade-Offs
"Pick two" in performance:
* Good
* Fast
* Cheap

"Pick two":
* High performance
* On-Time
* Inexpensive

Other trade-offs
* CPU and memory
    * Ex) Can use memory to cache results or CPU to compress memory

### Performance Tuning Efforts
Performance tuning targets, need to know which ones are worth the trade-offs:
| Layer | Example Tuning Targets |
| - | - |
| Application | Application logic, queue sizes, database queries |
| Database | table layouts, indexing,  buffering |
| System calls | Memory-mapped or read/write |
| File System | record size, cache size |
| Storage | RAID level, number and type of disk |

### Level of Appropriateness
* How deep to invest in a performance improvement depends on the organization
    * Is it a big org with a dedicated performance team?
    * Or a small org that rely on third parties to handle performance issues in products they use? Or only have time to tackle low-hanging fruit

### When to Stop Analysis
* When to stop analysis? When we found 1 reason for the slowdown? Or two? Or three?
    * No obvious answer
* May consider to stop analysis when:
    * When most of the performance issue is explained
    * When ROI is less than the cost to investigate
    * Bigger ROIs elsewhere

### Point in Time Recommendations
* Performance recommendations only usually apply for a *specific point in time*.
    * What improved performance a month ago may not be the same for the system today
* Workloads can change, more users, software/firmware changes, etc.
* Version control tuneable parameters with detailed history
