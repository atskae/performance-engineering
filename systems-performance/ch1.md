# Chapter 1: Introduction

## 1.1 Systems Performance
**Systems Performance**: Performance of the entire computer system:
* Software components
* Hardware components
* Anything in the **data path**
    * You should draw the data path of your application

Goal is to reduce computer latency and computing costs to improve the end-user experience.

## 1.2 Roles
Either each team has the responsibility to analyze performance of their own part of the stack (ex. networking, databases, cloud) or have *performance engineers*.

A **performance engineer** works with multiple teams and has an overview of the entire system
* Provide performance analysis tooling to other teams
* Each team member in a performance team can specialize (ex. kernel performance, language performance, tooling, etc)

## 1.3 Activities
Tasks of a performance engineer:
* Set performance objectives
* Prototyping
* Write benchmarks
* Monitoring performance of production software
* Look into incidents related to latency
* Write performance tools to help with analysis

**Capacity planning**: study the resource footprint of the software

## 1.4 Perspectives
Two important perspectives in performance analysis:
* Workload analysis
* Resource analysis

## 1.5 Performance Is Challenging
* Subjectivity: *"is this latency good or bad?"
* Complexity: many components interacting with each other
    * *Cascading failures*
    * Hard to reproduce latency issues of a component in isolation - only manifests when interacting with other components
    * Performance also depends on production workload - hard to reproduce in a test setting
* Multiple causes
* Prioritizing which performance issues are more important to address than others
    * Need to quantify the magnitude of each issue and compare the potential speedup of each fix
    * *Latency* is a metric used to quantify

## 1.6 Latency
**Latency**: measure of time spend waiting
* The time for any operation to complete (ex. webpage loading, database query)
* Allows for *speedup* to be measured
    * Measure best possible speedup if some latency was removed
    * Ex) A database query takes 100ms. 80ms of that is from disk reads. If we elimated disk reads (by caching), the database read would be 5 times faster (100ms - 80ms = 20ms), speedup = 5x

## 1.7 Observability
**Observability**: a way to understand the system through observation
* Ex) counters, profiling, tracing
* This does *not* including benchmarking, as it modifies the system's workload

### 1.7.1 Counters, Statistics, and Metrics
* Applications/kernels can provide data/metrics on their activity/state:
    * Ex) Operation counts, latency, error rates
* Can also read **counters** of events, which are usually incrementing and can be used to compute other metrics: ex) rate of change over time
* [Prometheus](https://prometheus.io/) is an open-source monitoring tool to track system metrics
* Metrics/visuals may only reveal that there is a problem but not the root causes - need profiling/tracing to dig deeper

### 1.7.2 Profiling
**Profiling**: using tools to *sample* measurements of a program

**Flame graphs** are an effective visualization of profiled metrics
* Easy to see time spent on lock contention, memory allocation, misconfigured networks

### 1.7.3 Tracing
**Tracing**: capturing event data and used later for analysis
* Can trace: system calls, network packets
* General-purpose tracing tools

**Static instrumentation**: trace points added to the source code

**Dynamic instrumentation**: adds instructions to a running program for tracing
* Similar to debuggers inserting a breakpoint

## 1.8 Experimentation
* Benchmarking tools
* Tools that generate synthetic workloads on a system measures the performance
* **Macro-benchmarks** simulate a real-world workload
* **Micro-benchmarks** test a specific component, such as CPU, disks, networks (ex. iperf TCP benchmark)
    * Fixed inputs to rule out network/client variance
* Best to start with observability tools on production systems first, then use experimental tools


## 1.9 Cloud Computing
* Can deploy new instances of compute on demand
    * Can look into how to do more with less compute resources (cost-savings)
* Look into performance effects by other tenants, *performance isolation*
* System observability by each tentant


## 1.10 Methodologies
* **Methodology**: documented steps for performing a task in systems performance (as opposed to trying out random things)
    * Example methodology: USE method
* Linux perf anaylsis in 60 seconds - checklist

Can use the following Linux commandline tools:

| # | Tool | Usecase | Output | MacOS |
| --- | --- | --- | --- | --- |
| 1 | `uptime` | How long the system has been up, number of users, and **CPU load**: how many processes are actively using/waiting for the CPU (1, 5, and 15 minute averages). A 1-core system at 1.0 load is 100% load. A 4-core system at 4.0 load is at 100% load. | ```16:44  up 11 days,  1:28, 2 users, load averages: 3.18 2.89 3.84``` | |
| 2 | `dmesg -T \| tail` | Shows the **kernel ring buffer**, which shows kernel log messages (ex. driver status, hardware errors) | | |
| 3 | `vmstat -SM l` | **Virtual memory statistics** (ex. pages) | | `vm_stat` |
| 4 | `mpstat -P ALL 1` | per-CPU balance | | `top -o cpu` |

... TODO add the rest.

## 1.11 Case Studies
**Case study**: a real example going through various tasks to investigate a performance issue
* When and why the tasks performed

### 1.11.1 Slow Disks
