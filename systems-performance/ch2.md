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
First check for errors - easy to interpret and objective. Then check for saturation.

Can find *a* bottleneck out of many potential bottlenecks.

(see USE method flow, p48)

**Expressing Metrics**
Express the main metrics:
* **Utilization**: percent over time interval (ex. *"1 CPU is running 90% utilization"*)
* **Saturation**: as a wait-queue length (ex. *"The CPU has an average queue length of 4"*)
* **Errors**: number of errors reported (ex. *"The disk drive has 50 errors"*)

* Possible to have bursts of short but high utilization, even if utilization over a longer period of time is lower
    * These short, high bursts can cause saturation on a system
    * Ex) toll booth is at 100% utilization when there are no more empty booths and cars need to start waiting in a queue (saturation)
        * The daily average utilization could be 40% but this does not reflect when/if saturation was reached within a day

#### Resource list
First step in the USE method: create a list of resources.

For example:
* **CPUs**: sockets, cores, hardware threads (virtual CPUs)
* **Main memory**: DRAM
* **Network interfaces**: Ethernet ports, Infiniband (networking standard used in HPC)
* **Storage devices**: Disks
* **Accelerators**: GPUs, TPUs FPGAs
* **Controllers**: Storage, network
* **Interconnects**: CPU, memory, I/O

Many types of resources:
* **Capacity resource**: main memory
* **I/O resource**: network interfaces (IOPS, throughput)
    * Also can be seen as a *queueing system*: resources that queue and then service these requests
* Both Capacity and I/O resource: storage device

Focus on resources that suffer under high utilization and saturation (for example, can keep hardware caches out of the resource list, since these caches are used to *improve* performance). In doubt, investigate the resource and see for yourself.

#### Functional Block Diagram
Draw a *functional block diagram* of the resources and their relationships with each other (see p50 for an example).

#### Metrics
For each resource, check: utilization, saturation, errors
* And metrics that capture those

Example table:
| Resource | Type | Metric |
| - | - | - |
| CPU | Utilization | CPU utilization (per CPU or system-wide average) |
| CPU | Saturation | Run queue length, scheduler latency |
| Memory | Utilization | available free memory |
| Memory | Saturation | Out of memory events, swapping |
| Network interface | Receive throughput / max bandwidth |
| Storage I/O | Utilization | Device busy % |
| Storage I/O | Saturation | Wait queue length |
| Storage I/O | Errors | Device errors |

See p51 for advanced USE metrics (harder to capture)
See Appendix A for USE method checklist for Linux

#### Software resources
* Locks, mutexes
    * Queued threads waiting for the lock
* Threadpools, processes, threads
    * Reaching maximum number of threads/processes
    * Errors "cannot fork"

#### Suggested Interpretations
* 100% utilization is usually a sign of a bottleneck
    * ~60% utilization can start to see queueing delays
        * Short busts of 100% utilization can be hidden in the average, so look closely
* Saturation, look at queue lengths over time
* Errors, increasing error counters

It is also worth confirming that utilization, saturation, and errors are low, then it could be possible to eliminate the problem as a *resource* problem and look elsewhere.

#### Resource Controls
**Software resource controls** on cloud computing and container environments to limit the resource use of each tenant on the system
* Ex) limit CPU/memory usage per tenant/application
* On Linux, `cgroups` are used to configure limits for resources

#### Microservices
* Can be a lot of metrics impossible to look through all manually

Example USE metrics for a Netflix microservice:
* **Utilization**: average CPU utilization across a cluster
* **Saturation**: look at 99th percentile latency (assume this is the point of saturation) vs. average latency
* **Errors**: request errors

[Netflix's Atlas cloude-wide monitoring tool](https://netflix.github.io/atlas-docs/overview/) observes these three metrics for each microservice at Netflix.


### The RED method
The USE method focuses on resources. The **RED method** focuses on services (cloud services in a microservice architecture).

The RED method defines three metrics defined from the user perspective. For every service, check the:
* **R**equest rate: the number of service requests per second
* **E**rrors: the number of requests that failed
* **D**uration: the time for a request to complete

Performance engineer task:
* Draw architecture diagram of all thes services
* Ensure the three RED method metrics are monitored for each service

The RED method was created by [Tom Wilkie](https://grafana.com/blog/the-red-method-how-to-instrument-your-services/), who implemented the RED and USE methods for the monitoring system [Prometheus](https://grafana.com/docs/grafana/latest/fundamentals/getting-started/first-dashboards/get-started-grafana-prometheus/) at Grafana.
* USE method for machine health
* RED method for user health

The *request rate* can reveal if the problem is with the software architecture or the workload.
* Steady request rate but increased request duration: architecture issue
* Both increase: workload issue, workload characterization needed

### Workload Characterization
**Workload characterization** tries to find the issues of the work applied to the system
* Focuses on the *input* to the system, rather than the resulting performance

Characterize workload by asking:
* **Who** is causing the load? A process? User? Remote IP?
* **Why** is the load applied?
* **What** are the load characteristics?
    * IOPS, throughput, reads/writes
    * Track variance (how far metrics are from the average/mean) and standard deviation (`sqrt(variance)`) when appropriate
        * Low variance - data is close to the mean
* **How** is the load changing over time? Daily patterns?

Always best to check metrics, even if you are confident in what to expect - there can always be surprises.
* Ex) You'd expect the only clients to a database app are web servers, turns out the the database is open to the whole internet and under a Distributed Denial-of-Service attack

Look for ways to eliminate *unnecessary work*, examples:
* Thread stuck in a loop, wasting CPU cycles
* Doing too many backups during peak hours

To **throttle** means to intentionally slow down speed/performance
* If the workload cannot be eliminated, can *throttle* the system using Resource Controls
    * Ex) Slow down backups so the production database is not overwhelmed during peak hours

Can use workload characterization to design **simulation benchmarks**
* Can use metrics like varianace/standard deviation of the workload to simulate all types of workloads (not just the average workload)
* See Chapter 12 on Benchmarking

Distinguishes load vs. architecture issues.

### Drill-down Analysis
**Drill-down Analysis** methodology starts with high-level metrics then digs deeper into software components that are relevant and eliminate areas that are uninteresting:
1. **Monitoring**: observe high-level statistics of the system over time, alerting if an issue occurs
    * Observability dashboard of all servers/cloud instances
    * **Simple Network Monitoring/Management Protocol** (SNMP) - historial tool to monitor devices attached to a network
    * *Exporters* are agents that run on a system and collect system metrics over-time, metrics to be viewed in a frontend/graphical interface
    * Find long-term behaviors over time
2. **Identification**: Narrow the issue to a specific resource or area of interest, identify the bottlenecks
    * Go onto the server directly and check system components: CPUs, disks, memory
    * Can use CLI tools (`vmstat`, `iostat`, etc) but can also use GUIs that already expose those metrics for faster analysis
3. **Analysis**: Further examination, find root-cause and quantify the issue
    * Look through traces
    * Inspection of source code

Netflix cloud example going through the drill-down methodology:
1. **Monitoring**: Look at Netflix Atlas, open-source cloud monitoring tool
1. **Identification**: Netflix perfdash - see the USE metrics of a single instance on a dashboard
1. **Analysis**: Netflix FlameCommander - create flame graphs, CLI tools over SSH (ex. `ftrace`-based tools)
    * Find where in the code that consumes a resource

### Five Whys
Can keep asking yourself ["Why?" 5+ times](https://en.wikipedia.org/wiki/Five_whys) to see if that reveals anything. Example:
1. The database began to perform poorly. Why?
1. Delayed due to disk I/O and paging. Why?
1. Database memory increased too much. Why?
1. Allocator is consuming more memory than usual. Why?
1. Allocator has memory fragmentation issue.


### Latency Analysis
**Latency analysis**: look at how long it takes for an operation to complete, then break this down into sub-components of where time is being spent
* Drill down through the software stack
```
|------------- Operation ---------------------------|
Latency(op) = Latency(A) + Latency(B) + ...
```

### Method R
Latency anaylsis on database queries

### Event Tracing
Systems operate by processing discrete events (ex. CPU instructions, disk I/O network packets):
* Might be necessary to dig deeper into these discrete events if the *summary* of these events (ex. ops/sec, bytes/sec, average latency) don't reveal enough
* Ex) network troubleshooting might require inspecting individual network packets `tcpdump`
* Ex) Storage device I/O `biosnoop`
* Ex) system call layer, tracing `strace`/`perf` on Linux
* **Latency outlier**: the high latency is caused by events before it, but not the event itself (ex. queueing)


### Baseline Statistics

Line graph, with x-axis as time.
* Can also graph this week's data and last week's data in the same line graph, to compare Tuesday with last week Tuesday at the same time

But there are other metrics collected at the command line that are not monitored
* *Baseline statistics* can be collected on a periodic schedule on a system

### Static Performance Turning
*Static performance turning*: look at how the architecture is configured
* As opposed to *dynamic performance*: the system when load is applied

Go through all the components in the system and ask:
* Is this still needed?
* Does the config make sense for the intended workload?
    * Ex. OS/firmware versions, network config (1 Gbits/s instead of 10Gbits/s)
    * Ex. Using a remote server for authentication vs. local server
* Any errors that resulted in a degraded state?
    * Ex. file system is full
    * Costly debug-mode accidentally left on

Easy to check, hard to remember to do them!

### Cache Tuning
* Aim to cache closest to the workload as possible
* Metrics: hit/miss rate
* Look for *double caching*: two caches consuming main memory but caching the same data

### Micro-Benchmarking
**Micro-benchmarking** test the performance of simpler/artificial workloads
* As opposed to **macro-benchmarking** (or *industry benchmarking*) which tries to test real-world workloads
    * Can be more complex to understand

Examples of micro-benchmarks:
* Linux `iperf`: TCP throughput test to catch networking bottlenecks

*Load generators* just generate small workloads (rely on other tools to capture metrics)
* *Micro-benchmarking tools* do both generate workload and compute metrics

Example targets of micro-benchmarks:
* System call time
* File system reads

* Conduct target operations quickly and computes the average time for each operation:
```
average_time = total_runtime / number_of_operations
```

### Performance Mantras
Tuning methodology on how best to improve performance, from most to least effective:
1. Don't do it: eliminate unnecessary work
1. Do it, but don't do it again: caching
1. Do it less: reduce polling, updates, refreshes
1. Do it later: write-back caching (cache writes, then write to slow-memory later)
1. Do it when they're not looking: schedule work to run during off-peak hours
1. Do it concurrently: single to multi-threade3d
1. Do it cheaply: Buy faster HW

[Scott Emmons](https://netflixtechblog.com/a-microscope-on-microservices-923b906103f4) at Netflix


## 2.6 Modeling
Many reasons to do *analytical modeling*:
* **Scalability analysis**: observing how performance scales as load or resources increase
    * Resources can be both hardware (ex CPU cores) or software (processes, threads)

Three main performance evaluation activities:
* Analytical modeling (scalability analysis)
    * Find when the performance stops scaling linearly and reaches a *knee point*: contention and performance degredation
* Observability of the production system (measurement)
    * Characterize load and resulting performance
* Experimental testing (simulation)
    * If the target production workload does not exist yet (not seen in production)

### Enterprise vs. Cloud
* With cloud computing, can re-create a production system and test there for the duration of a benchmark test
* Can re-create different production environments

### Visual Identification
* Can plot results and visually find where the knee point is
* ex. graph threads vs. throughput, and see when the slope starts to flatten (p.63)
    * Can see if the CPU core count and the number of hardware threads per core is related to the knee point
    * Test on different systems with different number of CPU cores to confirm a hypothesis

**Scalabilility profiles** are metrics plotted visually to observe changes to the system. The x-axis is the scalability dimension (listed below) and the y-axis is the resulting performance (throughput, transactions per second, etc.). Imporant scalability profiles to consider:
* **Linear scalability**: Performance increases proportionally as resources scales
    * Want to see at what point does the performance increase slows down
* **Contention**: Some shared resources can only be used serially
    * Too much contention can reduce the effectiveness of scaling
* **Cohherence**: Data coherence - keep track of data changes across the system, doing this at large scale mind outweigh the benefits of scaling
* **Knee point**: The point at which the scalability profile changes
* **Scalability ceiling**: A hard limit is reached (ex. reaching maximum throughput of a hardware resource or limits configured from resource control)

See p64 for example visual plots of the scalability profiles.

In addition to visual identification, mathematical models can be used.


### Amdahl's Law of Scalability
Computer architect Gene Amdahl proposed a formula to model system scalability:
* The model takes into account serial components that cannot be scaled in parallel.
* Used to studying the scaling of CPUs, threads, workloads

The model tries to measure contention on the serial resource:
```
C(N) = N / α(N - 1)
```
* `C(N)`: relative capacity
* `N`: scaling dimension (ex. CPU count, user load)
* `α` "Amdahl parameter" (where `0 <= α <= 1`): the degree of seriality
    * How it deviates from linear scalability

Steps to apply Amdahl's Law of Scalability:
1. Collect data to compute the range for the scaling dimension `N` of the system
    * Can use micro-benchmarking, load generators
2. Perform *regression analysis*, statistical method to estimate the relationship of a dependent variable with selected independent variables, to compute `α`
    * Can use statistical software (gnuplot, R)
3. Plot the data points visually to predict scaling
    * Find differences between data and model
    * Can also use gnuplot or R

### Universal Scalability Law
The **Universal Scalability Law** (USL), "super-serial model", developed by Dr. Neil Gunther
* Adds a parameter to include *coherence delay*

USL is defined as:
```
C(N) =  N / (1 + α(N - 1) + βN(N - 1) )
```
* `β`: coherence parameter
    * When `β == 0`, USL becomes the same as Amdahl's Law of Scalability
* Other variables are the same as Amdahl's Law of Scalability

Can plot both Amdahl's Law of Scalability and USL (see p66).


### Queueing Theory
**Queueing theory** (invented by Danish mathematician Agner Krarup Erlang ~1909) is the mathematical study of systems with queues
* Ways to analyze queue length, wait time (latency), utilization (time-based)
* Many SW and HW components can be modeled as *queueing systems*
    * Multiple queueing systems can form a *queueing network*
* Uses mathematics, statistics, probability to model queueing systems
* Erlang's C formula - calculates the probability that a caller must wait to reach an agent on the phone, given the number of agents and the traffic of calls
* Little's Law (John Little, was an operations researcher at MIT) - computes the average number of items/tasks in a system `L`:
    ```
    L = λ * W
    ```
    * `λ`: task arrival rate
    * `W`: the time it takes for a task to go through the entire system
    * Can be applied to a queue

Queueing systems can help answer questions such as:
* What happens to the mean response time when:
    * the load doubles?
    * an additional processor is added?
* Can we reach a 90th percentile response time <100ms if the load doubles?
* Utilization
* Queue lengths
* Number of jobs over time

Simple queueing model:
```
                           Queueing System
                ______________________________________
 Arrivals ---> |  Queue | | |  ----> Service Center   | ----> Departures
               ---------------------------------------
               |---wait time -|     |--service time --|
```

* Can have multiple service centers, also called *servers*, working in parallel

Can categorize queueing systems in three categories:
* **Arrival process*: arrival times to the queueing system, which can be random, fixed, Poisson (exponential distribution for arrival time)
* **Service time distribution**: service time of the *servers*/service center, can be fixed (deterministic), exponential, or other distribution types
* **Number of service centers**: one or many


#### Kendall's Notation
Queueing systems can be categorized and expressed in **Kendall's Notation** (by English mathematician and statistician David Kendall):
```
A/S/m
```
* `A`: arrival process
* `S`: service time distribution
* `m`: number of service centers
* Other variations of Kendall's notation adds: number of buffers in the system, population size, service discipline

Examples of commonly studied queueing systems:
* `M/M/1`: Markovian arrival times (exponential distributed), Markovian service times, one service center
* `M/M/c`: same as `M/M/1` but with multiple servers
* `M/G/1`: Markovian arrivals, general distribution of service times (any), one service center
* `M/D/1`: Markovian arrivals, deterministic service times (fixed), one service center

`M/G/1` is commonly used to study the performance of rotational hardware disks.

**Markov property**: future events only depend on the present state and not the past states before the present, also called "memoryless"/M in Kendall's Notation
* In queueing theory, having a *Markovian arrival rate* means each task arrives at random and the arrival rate of a single task is not influenced by the past tasks that came before
* *Exponential inter-arrival time*: short gaps between tasks happen more frequently, long gaps happen less frequently but still occur

#### `M/D/1` and 60% Utilization
A walkthrough of a simple example of a hard disk which follows the queueing model `M/D/1`, which means:
* has Markovian arrival times
* deterministic service times (this is obviously a simplification)
* 1 server

Question: how does the disk's response time vary as its utilization increases?

In queueing theory, the response time of a queueing system `M/D/1` can be computed as:
```
r = s(2 - p) / 2(p - 1)
```
* `r`: response time
* `s`: service time
* `p`: utilization

This formula can be graphed, for example, with service time `s` = 1ms and for utilization from 0% to 100% (see p68)
* Single service queue, constant service times (`M/D/1`)
* This can be plotted using R/gnuplot
* Can see where the response time quickly doubles, then triples
* Disk utilization can become a problem earlier before reaching 100% utilization
    * CPUs can pre-empt tasks for more important tasks, as opposed to hard disks, where all tasks must be queued


### 2.7 Capacity Planning
**Capacity planning**: examines how the system will scale as load increases
* Many ways to do capacity planning: modeling (described in previous section), looking at resource limits, factor analysis
* Solutions for scaling: load balancers, sharding
* A whole book on this topic: [The Art of Capacity Planning](https://dl.acm.org/doi/book/10.5555/3181173) (2017)


#### Resource Limits
Look for the resource that will become a bottleneck under load
* ex. a container reaches its resource limit configured by SW and becomes a bottleneck

Steps for resource limits method, measure and continuously monitor:
1. The rate of server requests over time
2. HW and SW resource usage
3. Express server requests in terms of resources used
4. Extrapolate server requests to known limits for each resource

First identify the type of requests that the server serves
* Web servers serve HTTP requests
* Network File System (NFS) serves NFS protocol requests
* Database serves query requests

Then compute the resource consumption per request
* Look at rate of requests and the resource utilization
* Can extrapolate to see which resource will hit 100% utilization first
* Future systems can use micro-benchmarks or load generation tools to compute these metrics
    * Existing systems can use actual client load / experimentally

Resources to monitor:
* **Hardware**: CPU utilization, memory usage, disk IOPS, disk throughput, disk capacity (volume used), network throughput
* **Software**: Virtual memory usage, processes/tasks/threads, file descriptors

Example walkthrough:
* We have a system that has a request rate of 1,000 requests per second
* Busiest resources are the 16 CPUs, with an average utilization of 40%
    * We predict that at 100% utilization we will encounter a bottleneck
* *What will be the request rate when CPU utilization is at 100%?*

What percentage of CPU does each request take?
```
16 CPUs * 40% / 1,000 requests = 16 * 0.40 / 1000 = 0.0064 = 0.64% CPU per request
```

Given 0.64% CPU per request, what is the request rate at 100% CPU utilization?
```
16 CPUs * 100% / x = 0.64% CPU per request
0.0064 * x = 16 * 1.0
0.0064 * x = 16
x = 2,500 requests per second
```

Of course, this is a rough estimate and other factors can contribute to the limiting factor before reaching this request rate.

We can also plot CPU utilization vs. throughput (request rate, requests/sec) with real data on a plot and extrapolate beyond a utilization % we haven't seen yet (see p71). We can then estimate the maximum possible throughput and improve our estimates over time with real data.

Given maximum throughput of 2,500 requests per second, we also need to ask *is this enough?*
* Need to understand peak workloads
* Sometimes peak workloads happen at various times (ex. new feature launch)


#### Factor Analysis
* Performance goal: achieve required performance for the minimum cost (in resources)
* Many factors to achieve this target performance
    * Varying number of disks/CPUs, RAM, RAID configs, etc.

Cannot test every possible combination of resources. One approach is to start with the maximum system configuration:
1. Test performance with the maximum resource limit(?)
2. Slowly reduce capacity of selected resources once by one - each change would degrade performance.
    * Keep track of this performance degredation as a percentage of peak performance from Step 1
3. Calculate the cost savings from reducing resource capacity
4. Compare with peak performance cost vs. reduced performance cost, try to maintain the required requests/sec of the system.
5. Retest delivered performance based on new resource configuration and compare the experimental performance with the calculated/predicted performance

**Example**: We have a new storage system with performance requirements:
* 1 GB/sec read throughput
* 200 GB **working set size**: the amount of actively used memory, frequently used within a time window
    * Ex. Device can be allocated 100 GB of memory but only ever work with 50 GB of memory on an active operation - 50GB is the *working set size*

The maximum configuration achieves:
* 2 GB/sec read throughput
* 4 processors
* 256 GB DRAM
* Two dual-port 10 GbE (Gigabit ethernet) **network cards**: also called a *network interface controller (NIC)*, HW that connects a computer device to a computer network
* Supports **jumbo frames**: network packets that are larger than the standard network packet size of 1,500 bytes
* No compression/encryption is enabled (costly to activate)

Let's say we experimentally reduce each resource and observed the following performance drops:
| Resource/Config Change | Performance Drop of Read Throughput |
| - | - |
| 4 processors to 2 | 30% |
| 2 network cards to 1 | 25% |
| Disable jumbo frames | 35% |
| Enable encryption | 10% |
| Enable compression | 40% |
| Use less DRAM (less caching) | 90% |

With these metrics, we can calculate a configuration that reduces cost but still meets the performance requirements.

For example, what is the throughput of the system reduced 2 processors with 1 network card?
```
2 processors * (1 - 0.30) * 1 network card * (1 - 0.25) = 1.05 GB/sec estimated
```
1.05 GB/sec meets the performance requirement of 1 GB/sec read throughput.
* Always validate the theoretical/calculated metrics experimentally and see how it compares
