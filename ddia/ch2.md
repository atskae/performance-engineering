# Chapter 2: Defining Non-functional Requirements

* *Functional requirements*: main functionality of the app: the task to accomplish, what screens/buttons to have
* *Non-functional requirements*: needs to be fast, performance, compliant with the law, secure, easy to maintain

The main non-functional requirements of this book:
* Performance: how to define and measure metrics
* Reliability: how can the service still be available if something goes wrong?
* Scalability: can easily add compute resources to handle more load
* Maintainability: easy to maintain longterm

## Case Study: Social Network Home Timelines
* Here is a simplified example of a social network like X (formally Twitter) (the book is up to date 😅)
* Overview: users can post messages and follow other users, assumptions:
    * ~500 million posts a day, ~5.8k-150k posts per second
    * Average user follows 200 users and has 200 followers

### Representing Users, Posts, and Follows
* What are the entities we need to represent in a database?
    * Separate database tables for: Users, Posts, Follow Relationships
* How do we build a *home timeline*: a page that shows recent posts from a user's followers?

* Want to fetch the recent posts that the user is following
    * If the followee (the person the user is following) makes a post, the user should see it immediately
    * Poll for new posts every 5 seconds
* If there are 10 million users online, 10 million / 5 seconds = 2 million posts per second
    * Expensive query, especially if a single user if following 200+ people
        * If a user has 200 followers, 2 million posts * 200 followers = 400 million queries per second

### Materializing and Updating Timelines
* How to improve from simply polling for new posts?
    * The server can immediately push new posts to the user
        * While user is logged out, pre-compute the home timeline
        * When user is logged in, user-side can subscribe to a stream of new posts
    * Caching
* Con of the above: more work when a user creates a new post
* **Fan-out**: when one requests results in multiple downstream requests
    * User makes a post -> fan out: each follower gets the new post
    * Let's say 5,800 posts per second are created
        * Each of these posts need to reach 200 followers
        * 5,800 * 200 = 1,160,000 writes per second
        * In comparison to 400,000,000 queries per second with polling

* **Materialization**: The process of pre-computing and updating the results of a query
    * **Materialied view**: a cached computed value
    * Speeds up reads, but more work on writes
* Other considerations
    * A user can't realistically read every post of every followee if that followee posts a lot
        * Can sample/drop some posts
    * Celebrity posts *cannot* be dropped
        * Celebrity users require more infrastructure (ref: "3% of Twitter's servers dedicated to Justin Bieber - 2010")


## Describing Performance
Two main metrics for software performance:
* **Response time**: The time it takes for a user to get a response after making a request
    * Unit: seconds, miliseconds, microseconds
* **Throughput**: Number of requests per second that the system is processing
    * Need to know max throughput of compute resources
    * Unit: "something" per second

Metrics of the Twitter case study example:
| Response Time Metrics | Throughput Metrics | 
| - | - |
| Time it takes to load home timeline, time until a post is delivered to users | Posts per second, timeline writes per second |

* Response time and throughput are related
    * Highly loaded system -> requests get *queued*, throughput goes down
    * If # of request reaches hardware max capacity, response time goes up significantly
    * See graph p37

* Users care about response time
* Throughput is related to how much computing resources are needed
    * Ex) The cost of a workload is how many servers you need to handle that workload
    * A system is *scalable* if the max throughput can increase by addeding more compute resources

### When an Overloaded System Won't Recover
* When the system is overloaded, queue is slow and throughput goes significantly down
* Response times for users go up, which makes users retry their request, which increases the load
    * **retry storm**: When retry requests overload the system
* Might require a system reset to recover - **metastable failure**
    * Causes a serious outage in prod
* Ways to avoid a *retry storm*:
    * Add random, longer waits in the client before retrying again, *exponential backoff*
    * If the service is overloaded, the client can immediately return an error
        * **circuit breaker**: detect a failure and prevent the failure from cascading to new failures
        * **token bucket** algorithm: a way to keep a long-term constant rate of accepting requests
            * Rate-limiting/controller
            * Tokens keep track of how many requests the system can make
            * Tokens accumulate at a fixed rate into a "bucket" - if the bucket is full, no more tokens are added
            * A token is used when a system handles a request
        * **load shedding**: proactively reject requests when the system is *close* to reach overload
        * **backpressure**: notify the client to stop sending requests since the system is overloaded
        * Explore load-balancing and queueing algorithms for different use-cases

## Scalability

### Understanding Load

Questions to ask about load:
* What happens if our load doubles?
* What is the peak load? (ex. maximum number of users online simultaneously)
* Peak memory?
* Throughput: requests/second, GB of memory used per day
* Ratio of reads to writes
* What is the average case? Extreme cases? Are we bottle-necked by the small, extreme cases?

When increasing the load, two ways to look at it:
* If load is increased, how much does performance get affected?
* If load increased, how much resources do we need to add to keep performance the same?

Two goals to reach:
* Maintain the performance written in the SLA (Service Level Agreements)
* Minimize cost of resources (CPUs, memory, network bandwidth)

### Shared-Memory, Shared-Disk, and Shared-Nothing Architectures
* *Shared-memory architecture*: run on a single machine
    * Takes advantage of threads of the same process being able to access the same RAM
    * Twice the hardware resources actually costs more than twice the cost of a lower spec machine
    * Usually unable to handle twice the load
* *Shared-disk architecture* - use several independent CPUs and RAM, then data is stored among shared array of disks
    * The CPUs share access to the disks
    * Disks are connected with *network-attached storage* (NAS) or *storage-area network* (SAN)
    * Contention/locking limits the scale of this architecture
* Shared-nothing architecture* - separate nodes, each node has their own CPUs/RAM/disks, and interact with other nodes at the software level
    * A distributed system
    * Adding more nodes is called **horizontal scaling**, as opposed to **vertical scaling** (add more CPUs/RAM/disk to 1 machine/node)

Pros of shared-nothing architecture:
* Potential to scale linearly
* Distributed across geography (if an outage in 1 cluster, other clusters are still up) - better fault tolerance
* Easily add/remove compute resources/nodes when load changes

Cons of shared-nothing architecture:
* Complexity of distributed systems
* Sharding

### Principles for Scalability
* Architecture of choice depends on the application (no 1 size fits all)
* Different scenarios: 100k requests per second, each 1KB, has different needs than 3 requests per minute, each 2GB
    * Both have same throughput: 100MB/sec
* The same architecture might be ideal at some levels of load, but might need to be completely rethought for a different order of magnitude of load
    * Experiment and evole the product over time - sometimes premature optimization before learning how the sysetm handles load is unproductive
* Focus on what services can be run independently of each other (microservice architecture)
    * Easier to scale
    * But this requires knowledge on the balance of keeping services together / separately
* Don't overcomplicate
    * If a single machine architecture is fine, don't spend time on a complicated distributed system
    * Autoscale systems might be overkill if the load is always predictable (manual load balancing might be better)
