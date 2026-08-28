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


## 2.4 Perspectives
Two common perspectives for performance analysis:
* *Workload analysis* (top-down, relative to the figure below)
* *Resource analysis* (bottom-up)

Figure:
```
Workload
  |
  v
Application
  |    |
  |    v
  |  System Library
  |
  v
System Calls
  |
  v
Kernel
  |
  v
Devices
```

### Resource Analysis
* *Resource analysis*: analysis of system resources: CPU, memory, disks, network interface, buses, interconnects
* *System administrators* look into whether the resources are a cause of performance issues, or for capacity planning for new systems
* Metrics:
    * IOPS
    * Throughput
    * Utilization
    * Saturation
    * Latency - given the workload, was is the latency of the resource to respond
* CLI "stat" tools: `vmstat`, `iostat`, `mpstat`

### Workload Analysis
* *Workload analysis* studies the performance of applications
* Look at workload applied and how the application responds
```
                       -------------
Input (workload) ---> |             |
                      |             |
         Latency      | Application |
                      |             |
Completion <--------  --------------
```
* This analysis is done by the software developers of the application and those who configure it
* Targets:
    * **Requests**: the workload applied
    * **Latency**: the response time of the application
    * **Completion**: looking for errors

* **Workload Characterization**: looking at the atttributes of the workload
    * Database example attributes: client host, tables, query string
    * Try to find unnecessary/unbalanced work
    * Even if the system is performing with low latency, we still want to elimiate any unnecessary work: "The fastest query is the one you don't do at all"

* Metrics for workload analysis:
    * Throughput (transactions per second)
    * Latency


## 2.5 Methodology
(big table of methodologies and anti-methodologies, p40)

### Streetlight Anti-method
Simply look around comfortable tools or random tools from the internet in hopes to find something obvious
* Tuning random configurations in hope that something will reveal itself
* Can easily overlook many issues
* Even if this method reveals *an* issue, it may not find *the* issue
    * And it's slow - there are faster, more smarter ways
* *Streetlight effect*: we look for solutions at places that are easy to see, not where the truth actually is

### Random Change Anti-method
Change random parameters, then measure performance
* If it was better than baseline, keep the new parameter
* It is time-consuming and may not be an ideal configuration long-term

### Blame-Someone-Else Anti-method
* Find a component that you are not responsible for, then make *that* team do the performance analysis
    * You only hypothesize the problem could be somewhere, then make another team do the work
* If blamed, ask the requester for screenshots/tools/data pointing the problem to your team first
* Don't waste other team's time on performance analysis without doing your own anaysis first

### Ad Hoc Checklist Method
* A checklist to find common peformance issues
    * Ex) after a new deployment, run `iostat` -> `r_wait`
* Remember to keep the checklist updated
* Helps to have a documented checklist so everyone knows how to handle common issues

### Problem Statement
Define the problem statement by asking questions:
* What makes you think there is a performance problem?
* Has the system ever performed well?
* What changed recently? SW? HW? Load?
* Can the problem be expressed in terms of latency or runtime?
* Does the problem affect other people or other application? (or just you?)
* What is the environment? What SW/HW is used? Versions? Configurations?


### Scientific Method
Make a hypothesis and test it. The general steps:
1. Question: the performance problem statement
2. Hypothesis: what do I think is the cause of the performance issue
3. Prediction: what I think will happen if I conduct the tests
4. Test: perform the test
    * *Observational test*: look at performance metrics of two different systems for comparison (ex. cache hit rate)
    * *Experimental test*: make a change on the system (ex. increase cache size)
5. Analysis: analyze the data collected from the tests

Example problems/walkthroughs on p45

* *Negative test*: intentionally hurting performance (ex. choose to reduce cache size) to learn more about the system

### Diagnosis Cycle
Iterative scientific method essentally:
```
hypothesis -> instrumentation -> data -> hypothesis
```
* Use when we can quickly get data on a new hypothesis and can iterate on a new hypothesis based on previous results

### Tools Method
A tools-oriented approach:
* Make a list of available tools
* List the metrics extracteed from each tool
* Explain how each metric can be interpreted

Can be prescriptive and the user is unaware that the available tools do not give a complete picture of the whole system

### The USE Method
* For every system resource, check the: **U**tilizaiton, **S**aturation, **E**rrors
* Should be done early in performance analysis

Terms:
* **Resource**: physical server functional components (Ex. CPUs, buses)
* **Utilization**: the percentage of time in a time range that the resource was busy servicing work
    * Can still accept more work while doing work, until the resource becomes *saturated*
    * *Capacity-based* (ex. how much memory was used in main memory)
    * *Time-based* (how long was the resource busy)
* **Saturation**: when the resource has extra work that it cannot service (ex. work waiting on the queue), also called *pressure*
* **Errors**: error events

* Contrast to the Tools method, the USE method iterates on system resources rather than performance tools
    * Helps come up with the questions to ask, then use tools to target the analysis to more specific metrics

**Procedure**
1. Check for errors - easy to interpret and objective
2. Check for saturation

Can find *a* bottleneck out of many potential bottlenecks.

**Expressing Metrics**
Express the main metrics:
* **Utilization**: percent over time interval (ex. *"1 CPU is running 90% utilization"*)
* **Saturation**: as a wait-queue length (ex. *"The CPU has an average queue length of 4"*)
* **Errors**: number of errors reported (ex. *"The disk drive has 50 errors"*)
