A unique ID helps us identify the flow of an event in the logs and is useful for debugging. A real-world example of unique ID usage is Facebook’s end-to-end performance tracing and analysis system, Canopy. **Canopy** uses TraceID to identify an event uniquely across the execution path that may potentially perform hundreds of microservices to fulfill one user request.
## Requirements for unique identifiers

- **64-bit numeric ID**: We restrict the length to 64 bits because this bit size is enough for many years in the future. Let’s calculate the number of years after which our ID range will wrap around.
    Total numbers available = 2^64 = 1.8446744 x 10^19
    Estimated number of events per day = 1 billion = 10^9
    Number of events per year = 365 billion = 365×10^9
    Number of years to deplete identifier range = 2^64 * 365×10^9​ = 50,539,024.8595 years

64 bits should be enough for a unique ID length considering these calculations.

## Solutions:

### 1. UUID

(+) doesn’t require synchronization between servers.
(+) simple
(+) scalable
(+) available 

(-) not really unique: UUID collision probability is low when there is sufficient randomness while implementing the UUID generator. 122 bits of UUID version 4 are set using a random or pseudo-random data source. If the randomness of such a data source is low, the collision probability can increase quite a bit.
(-) might not be monotonically increasing

### 2. DB

(-) single point of failure
(-) problems with consistency: The task of adding and removing a server can result in duplicate IDs. For example, suppose `m=3`, and server A generates the unique IDs 1, 4, and 7. Server B generates the IDs 2, 5, and 8, while server C generates the IDs 3, 6, and 9. Server B faces downtime due to some failure. Now, the value `m` is updated to 2. Server A generates 9 as its following unique ID, but this ID has already been generated by server C. Therefore, the IDs aren’t unique anymore.

(+) scalable

### 3. Range handler

Let’s try to overcome the problems identified in the previous methods. We can use ranges in a central server. Suppose we have multiple ranges for one to two billion, such as 1 to 1,000,000; 1,000,001 to 2,000,000; and so on. In such a case, a central microservice can provide a range to a server upon request.

This resolves the problem of the duplication of user IDs. Each application server can respond to requests concurrently. We can add a load balancer over a set of servers to mitigate the load of requests.

We use a microservice called **range handler** that keeps a record of all the taken and available ranges. The status of each range can determine if a range is available or not. The state—that is, which server has what range assigned to it—can be saved on a replicated storage.

This microservice can become a single point of failure, but a **failover server** acts as the savior in that case. The failover server hands out ranges when the main server is down. We can recover the state of available and unavailable ranges from the latest checkpoint of the replicated store.

(+) scalable
(+) available
(+) no duplicates
(+) numeric

(-) we lose a significant range when a server dies and can only provide a new range once it’s live again
![](Pasted%20image%2020240127220104.png)

# Unique IDs with Causality



# References:

1. [Как генерируются UUID](https://habr.com/ru/company/mailru/blog/522094/)
2. [Design Unique ID Generator](https://ishan-aggarwal.medium.com/design-unique-id-generator-c350281382d3)
3. [Design Unique ID Generator In Distributed Systems](https://medium.com/@samjingwen/design-unique-id-generator-in-distributed-systems-1e0f15f8b46e)
4. [How to Generate Unique IDs in Distributed Systems: 6 Key Strategies](https://levelup.gitconnected.com/how-to-generate-unique-ids-in-distributed-systems-6-key-strategies-37a8ab3b367d)