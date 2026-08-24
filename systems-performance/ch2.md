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

### Load vs. Architecture
* *Architecture*: software implementation, hardware configuration
    * Issues: single-threaded application, requests still get queued even if other CPUs are available but idle
    * Multi-threaded application that contends for a single lock, causing other users to wait
* *Load*: too much work load applied to the system can result in long queue waits and latencies
    * Issues: all CPUs are occupied but there is still queued jobs (more load than CPUs can handle)

### Scalability
* **Scalability**: the performance of a system as workload increases
    * Load vs. throughout graph
* Ideally expect *linear scalability*: as load increases linearly, so does throughput
    * But in real system we reach a *knee point*, where the throughput stops being linear and flattens or worsens due to resource contention
    * **Saturation point**: 100% utilization occurs
* Knee point vs saturation point
    * Knee point: when performance starts to degrade
    * Saturation point: the best throughput that can be achieved from the system - beyond this point, performance degrades and more errors occur
* Can also look at: # CPUs vs. throughput, load vs. response time
* Linear scalability and response time
    * Can start to return 503 errors instead of queueing requests to keep response time consistent

### Metrics
* **Metrics** are statistics generated by the system, applications, and tools, that measure activity of interest
* Performance metrics:
    * Throughput: operations/data volumes per second
        * Depending on context, the specifics of throughput changes:
            * Databases: queries/requests per second
            * Network: bits/bytes (volume) per second
    * IOPS: input/output operations per second (reads and writes)
    * Utilization: how busy a resource is, as a percentage
    * Latency: Operation time as an average or percentile

**Overhead**
* **Observer effect**: Gathering performance metrics also has a performance cost to gather and store the metrics

**Issues**
* There can always be bugs in the metrics-collecting itself
    * A new version of the software could introduce errors in metrics collecting
* Metrics can be confusing, complicated, unreliable
* Always need to validate metrics collecting - never assume it is always working correctly!

### Utilization
* Utilization can be time-based or capacity based
* In OS, can mean the usage of CPU and disk devices

**Time-Based**
* **Time-based utilization**: the amount of time the resource was busy

```
U = B/T
```
```
U = utilization
B = total time the system was busy
T = total observation period
```
* OS command line performanec tools compute utilization (ex. `iostat`)
* Utilization tells us how busy a resource is. At 100% utilization, we expect to see a performance degredation
    * Need to do further analysis if this becomes a serious bottleneck
* A resource can still be at 100% utilization but can still accept more work
    * Example: an elevator is 100% utilized but can still pick up people on the way if there is space
        * The focus here is the time that the elevator is being used
    * A disk at 100% utilization can buffer reads/writes to be completed later

**Capacity-Based**
* **Capacity-based utilization**: Proportion to the current amount of work the resource is completing to the ideal maximum amount of work the resource can perform
    * 100% capacity-based utilization: the disk cannot accept more work
    * 100% time-based utilization: the disk is busy doing work 100% of the time
    * 100% busy does *not* mean 100% capacity
        * An elevator that is completely full is at 100% capacity and cannot accept any more people

* The time-based utilization is also called *non-idle time*
* Capacity-based utilization is usually in the context of memory usage, data volume

### Saturation
* **Saturation**: when more work is requested of a resource that can be processed
    * Begins at 100% capacity-based utilization - no more work can be accepted and becomes to queue

### Profiling
* **Profiling** a system: periodically sampling the system state

### Caching
* **Caching**: storing results in a faster-access memory than slower-access memory
* Cache metric: **cache hit ratio**:
```
cache hit ration = hits / total_accesses
cache hit ratio = hits / (hits + misses)
```
* Higher hit ratio means the needed data was obtained via fast-access memory, improving performance
* Hit Ratio vs. Performance is nonlinear due to difference in speed between cache hits and misses (p.36)
    * The performance difference between 98% and 99% is much greater than 10% and 11%
* Multiple tiers of caches, each becoming larger, slower, cheaper (ex. L1, L2, L3 caches, main memory, disk)
* **Cache miss rate**: cache misses per second
* Need to look at both the cache hit ratio and cache miss rate
```
runtime = (hit rate * hit latency) + (miss rate * miss latency)
```

**Algorithms**
* Most-recently used (MRU): *cache rention policy*, keep recently used data in the cache
* Least-recently used (LRU): *cache eviction policy*: evict least-recently used data from the cache if cache is full

**Hot, Cold, and Warm caches**
* Cold cache: empty cache, cache with no useful data
    * 0% hit ratio
* Warm cache: contains data but not high enough hit ratio
* Hot cache: populated with commonly-requested data, high hit ratio

### Known-Unknowns
* **Known-knowns**: Metrics/rituals that we already know about
    * Ex. we already collect utilization metrics, we know this system is at 10% capacity
* **Known-unknwons**: Things we know to do but haven't done yet
    * Ex. We know we need to apply profiling to an application but we don't have that yet so we don't know what the sampling will result to
* **Unknown-unknowns**: Don't know that we do not know.
    * Ex. No idea that device interrupts are causing huge slowdowns, but we don't observe them/unaware of them
* The more we learn about systems, the more known-unknowns we learn about, and we can check them in our applications
