First let’s define the primitives. They are also known as threats.
- **Error**: Incorrect result (a.k.a Mistake)
- **[Fault](Fault%20Tolerance.md)**: Incorrect state/code (a.k.a. Defect)
- **[Failure](Failover.md)**: Observable incorrect behaviour

=> an Error can cause a Fault which might end up in a Failure.

### Quality Attributes

Now we know the basics, let’s see what sort of concepts can we derive from them.
- **[Availability](Availability.md)**: Probability of a system to operate in a given uptime
- **[[Durability]]**: Ability of a system to operate for a given (long) period without interruption
- **[[Robustness]]**: Ability of a system to handle erroneous inputs and errors during execution
- **[Fault Tolerance](Fault%20Tolerance.md)**: Property of a system to continue operating in case of failure
- **[[Disaster Recovery]]**: Set of tools, procedures to recover after failure

As you can see the whole spectrum / process is covered: from healthy operation through undesired failures till full / partial recovery.
### Compound Quality Attributes

Whenever we talk about things, like reliable systems, then we actually think about a group of attributes.
- **[Reliability](Reliability.md)**: Ability to operate all the time
    - This is the desired outcome
- **[Robustness](Robustness)**: Ability to respond for all input
    - Whatever happens it sends a response to the requester
- **[Resilience](Resilience.md)**: Ability to respond for environment changes
    - Withstand certain types of failure and remain functional
### Expect the unexpected

When the system has to face with unexpected behaviours we can distinguish several strategies.
- **[[Fault Tolerance]]**: Ability to survive misbehaving environment
- **[[Fault Resilience]]**: Ability to recover from misbehaving environment
- **[[Antifragility]]**: Ability to resist misbehaving environment and then get better

## Taxonomy

- **Prevention**: Something bad is going to happen and we can cease it before it occurs
- **Deterrence**: If Something bad is going to happen and we will impose a penalty
- **Detection**: Something bad is happening and we are able to notice it
- **Mitigation**: Something bad is happening and we are able to reduce the impact

![](Pasted%20image%2020240804232825.png)

## Basic mechanisms

### Fail Fast

**Categories**: proactive, before the fact

> Please bear in mind that fail fast is not equivalent with fail safe! The later one is not a robust solution because it [conceals the defect](https://javapapers.com/core-java/fail-fast-vs-fail-safe/).

If we know that it will fail because some of the circumstances aren’t met then we should exit early.

**Application**: This principle can be used on almost every level:
- A function should check its parameters and if they are not in the desired state then it should early exit
- An endpoint should check the requester’s rights and if they are not sufficient then the request must be rejected
- A service instance should check its dependencies' availability and if they are not reachable then the load balancer should not direct traffic to this instance

### Escalation

**Categories**: reactive, after the fact

**Application**: Even though this principle is really general it is not applied everywhere:
- On a function level, we can rarely find things like hierarchy, except the call stack. There are languages (like golang) where there are [defer statements](https://blog.golang.org/defer-panic-and-recover) and they were designed for clean-up / recovery. (It is a more general concept than the `finally` block).
- On application level there are several concurrency models (like [actor model](https://en.wikipedia.org/wiki/Actor_model), [communicating sequential processes](https://en.wikipedia.org/wiki/Communicating_sequential_processes)) that are heavily relying on [supervision](https://getakka.net/articles/concepts/supervision.html).

### [[Fallback]]

**Categories**: reactive, after the fact

> Please note that _fallback_ and *fail back *are not the same. The former one is more general and it means using a surrogate option. The latter one is used during **disaster recovery**:
> 
> - when we switch from the unhealthy machine to the backup it is called _fail over_
> - when we switch back from the backup to the healthy machine it is called _[fail back](Failover.md)_

This is the simplest pattern among all. If a given component is unable to processed with its normal flow (for whatever reason) it can return with a predefined (rarely dynamically constructed) “basic“ response in order to improve robustness. Remember robustness means, whatever happens the requester will receive some sort of answer. This answer is usually a reference to a value which is outside of the normal flow. Most of the time this pattern is applied as a last chance. If none of the ancestors in the supervision hierarchy are able to handle the problem then we might end up in a situation when we should fall back to our last resort. In other words, this pattern is rarely used alone, it is mostly the last item in the chain of the resilience strategies. A common pattern is the combination of _[caching](0.%20Cache.md)_ & fallback: _If I can’t use fresh data then I will fall back to use stale one_.

**Application**: This simple pattern appears on each and every level
- On programming language level we can use things like [default value](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/default) expressions, [conditional operator](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/conditional-operator) (`?:`) and [null coalescing operators](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator) (`??` | `??=`)
- On BCL level you can find a lot of methods that accept fallback value (a.k.a default), for example [GetValueOrDefault](https://referencesource.microsoft.com/#mscorlib/system/nullable.cs,61), [FirstOrDefault](http://www.technicaloverload.com/linq-single-vs-singleordefault-vs-first-vs-firstordefault/)
- On function level we can define a response variable with an extremal value. If everything works fine then we will overwrite that, otherwise we will use the initialization value
- In case of inheritance we rely on this simple principle as well. If the derived type does not override a given function then it will fall back to base class’s implementation
- …
- On infrastructure level this mechanism is heavily used to provide high availability, for example using replication in order to overcome on a single instance malfunctioning / failure.
    - If a given instance is not reachable then we can fall back to the other instance

### [[Retry]]

**Categories**: reactive, after the fact

> The relation between retries and attempts: n retries means at most n+1 attempts. The +1 is the initial request, if it fails (for whatever reason) then retry logic kicks in. In other words, the 0th step is executed with 0 delay penalty.

There are situation where your requested operation relies on a resource, which might not be reachable in a certain point of time. In other words there can be a temporal issue, which will be gone sooner or later. This sort of issues can cause **transient failures**. With retries you can _overcome_ these problems by attempting to redo the same operation in a specific moment in the future. To be able to use this mechanism the following criteria group should be met:
- The potentially introduced observable impact is acceptable. 
	- The word failure indicates that the effect is observable by the requester as well, for example via higher latency / reduced throughput / etc.. If the “penalty“ (delay or reduced performance) is unacceptable then retry is not an option for you.
- The operation can be redone without any irreversible side effect. 
	- This requirement is also known as idempotent operation. If I call the action with the same input several times then it will produce the exact same result. In other words, the operation acts like it only depends on its parameter and nothing else influences the result (like other objects' state).
- The introduced complexity is negligible compared to the promised reliability. 
	- This condition is even though one of the most crucial, this is the one that is almost always forgotten. As always there are trade-offs (If I introduce Z then it will increase X but it might decrease Y) and we should be fully aware of them.
    - Unless it will give us some unwanted surprises in the least expected time.

**Application**: This strategy is mainly used on higher levels even though it tries to overcome the lower levels' unreliability.
- On component level the retry logic is normally used during external resource communications:
    - Http (overcome network unreliability)
    - Database (overcome network unreliability and deadlocks)
    - etc.
- …
- On infrastructure level we can use a [service mesh](https://en.wikipedia.org/wiki/Service_mesh) implementations, where its [data plane](https://en.wikipedia.org/wiki/Data_plane) component transfers the traffic through a proxy and its [control plane](https://en.wikipedia.org/wiki/Control_plane) part can enforce the retry policy. 

**Parameters**:
- **Trigger**: When it should be fired? How is it determined that a previous call failed?
- **Count**: How many attempts should be done at most before the operation is treated as failed request?
- **Delay**: How much time should be waited before the next attempt?
- **Strategy**: What sort of strategy is used to calculate the delays? Are they the same always? Are they increasing each and every time, like exponentially? etc.
- **Jitter**: Should it rely on some sort of randomness to prevent a burst?

### [Circuit Breaker](Circuit%20Breaker.md)

**Categories**: proactive, before the fact

> Circuit breaker detects failures and prevents the application from trying to perform the action that is doomed to fail (until it's safe to retry) - [Wikipedia](https://en.wikipedia.org/wiki/Circuit_breaker_design_pattern)

So this tool works as a mini _data_ and _control_ plane. The requests go through this proxy, which examines the responses (if any) and it counts subsequent failures. If a predefined threshold is reached then the transfer is suspended temporarily and it fails immediately.

**_Why is this useful?_**

Because it prevents [cascading failures](https://en.wikipedia.org/wiki/Cascading_failure). In other words the transient failure of a downstream system should not be propagated to the upstream systems. By concealing the failure we are actually preventing a chain reaction (domino effect) as well.

**_How does it know when a transient failure is gone?_**

It must somehow determine when would be safe to operate again as a proxy. For example it can use the same detection mechanism that was used during the original failure detection. So, it works like this: after a given period of time it will not block the next incoming request and it examines its result. If it succeeds then the downstream is treated as healthy. Otherwise nothing changes (no request is transferred through this proxy) only the timer is reset.

**_What states does it use?_**

The circuit breaker can be in any of the following states: `Closed`, `Open`, `HalfOpen`.

1. `Closed`: It allows any request. It counts successive failed requests.
    1. If the successive failed count is below the threshold and the next request succeeds then the counter is set back to 0.
    2. If predefined threshold is reached then it transitions into `Open`
2. `Open`: It rejects any request immediately. It waits a predefined amount of time.
    1. If that time is elapsed then it transitions into `HalfOpen`
3. `HalfOpen`: It allows only one request. It examines the response of that request:
    1. If the response indicates success then it transitions into `Closed`
    2. If the response indicates failure then it transitions back to `Open`

After each state transition all counters are set to zero and all timers are reset.

**Application**: The circuit breaker application area is the same as for the retry. Why? Because most of the time they are used together. For example there is a retry logic with 5 retries and a circuit breaker with 3 subsequent failures. After the 3rd failed attempt the circuit breaker opens and prevents the 3rd retry. It waits a bit and moves into the Half Open state. When 4th retry kicks in then depending on its result the circuit breaker moves into Closed or Open state.

**Parameters**:
- **Failure detection**: How do we define that a request does not succeed?
- **Failure count**: How many successive failed requests should be awaited before the breaker opens?
- **Failure penalty**: How much time should be waited before the next request is permitted?

Alternatively the failure count can be defined as a percentage rather than a concrete value.

You can also examine only every nth request, so you can use sampling to reduce the overhead.

### [Throttling](1.%20Software%20Engineering/2.%20Architecture/Examples/Components/Rate%20Limiting.md)

**Categories**: proactive, before the fact

> This pattern is also known as [Rate Limiting](1.%20Software%20Engineering/2.%20Architecture/Examples/Components/Rate%20Limiting.md).

The [Circuit Breaker](Circuit%20Breaker.md) and [Retry](Retry.md) logic are patterns that are used on the consumer-side, while throttling is used on the provider-side. The provider can be easily overwhelmed / overloaded by flooding it with tons of requests (see [DOS and D-DOS attacks](https://en.wikipedia.org/wiki/Denial-of-service_attack)). Even though there are auto-scaling rules, the server resources are not unlimited. So, we as service developers have to protect our valuable resources in order to sustain our [SLA / SLO](Incident%20response%20process.md)

Usually we can distinguish two kinds of traffic curve. There is one which can be seen on a daily basis, which has known warm-up time period(s) and some short spikes. The other kind is when the traffic curve is wandering around the available resource limit and it is sustained for a fairly long period. What can we do? There are several options like adding new resources; queuing the requests; rejecting requests; etc..

Throttling is all about limiting the rate of the incoming requests by rejecting some. It is usually done by counting requests based on some criteria and when a certain threshold is reached then the requests are rejected for a predefined period of time. When that period is elapsed then the requests are accepted again. Can you see the resemblance with the circuit breaker? It is not a coincident, you can think of it as its counterpart.

**Application**: This pattern is mainly applied on fairly high levels because it uses the fail fast principle as close as possible to the entry points of the service.
- On the API level, the controllers / actions can be decorated with this policy. Usually the counting is based on the callers IP-address.
    - It is really common to introduce several layers of thresholds to have really fine-grained control over the requests. For example:
        - 5 requests / min
        - 100 requests / hour
        - 1000 requests / day
- In case of multi-tenant application there is a dedicated layer called governance, which is responsible to avoid resource abuse.
    - The limits are counted on tenant basis and sometimes these thresholds are
        - **soft**: an increase can be requested through support
        - **hard**: when it is reached then penalty is imposed
- On the cloud provider (can be considered as giant multi-tenant service provider) level different services can apply different penalties
    - For example:
        - **API gateways**: the instances are auto-scaled and the consumer is over-charged
        - **Databases**: if the provisioned read/write is reached then the previously “saved up” capacity can be burned in order to sustain short bursts

### Bulkhead

**Categories**: proactive, before the fact

> The name of this pattern comes from the naval ship design. A bulkhead is a dividing wall / barrier between other compartments. For example if a section is damaged then only that compartment is compromised and others remain intact.

Which means **a single crack won’t result as a total failure of the whole system**. That’s why this sectioned partitioning technique can be really useful for distributed systems as well.

This pattern can be used on the provider side as well, just like the throttling. It can be used to isolate resource pools and prevent resource abuse. Consider the following situation:
- Your service stores and retrieves data to and from its persistent store
- You can divide the collaborating services as either provider (causes write) or consumer (causes read)
- How could you make sure that neither the provider nor the consumer side can not cause starvation for the other side?

**_What does starvation mean?_**

This describes a situation when the resource reader/writer is unable to gain regular access to the shared resource and because of this it is unable to make progress

**_When can this occur?_**

When one of the resource users overuses the shared resource and causes others to wait (or even worse timeout)

**_How does this relate to deadlock and livelock?_**

- **[Deadlock](1.%20Software%20Engineering/3.%20Database/OTLP/SQL/3.%20Transactions/Transaction%20Locks/_Base.md)**: Execution is suspended because there is a cyclic dependency between them so no progress has been made.
- **Livelock**: Execution continues but each participant is occupied by reacting to other participants' action so no actual progress has been made due to extensive cross-communication.
- **Starvation**: Execution continues because there is no cyclic dependency, but the resource scheduler’s queue is growing. There is a small progress even though utilization is high, the throughput is low.

**_What can we do to prevent starvation?_**

- We can create separated bounded pools.
    - With separation we gain isolation, which means less interference.
    - With bounds we gain prevention against overage.
        - These bounds can be static or even dynamic to adjust to current workload

**Application**: This pattern can be used on each and every level where we want to protect our expensive resources
- On thread pool level you can divide the pool into smaller isolated thread pools to rate limit different background processes
    - Being able to control _noisy neighbours_
- On database connection pool level you can divide your connections based on the workload (for example: 50% query, 10% replication, 10% write, 30% buffer)
    - Being able to support _unusual demand_
- On network communication level you can use isolate service failures from other service communication
    - Being able to mitigate _failure propagation_
- On tenant level we can use this pattern to dedicate more resources for our paid users than to our normal free users
    - Being able to handle _priority consumers_

### Load Shedding

**Categories**: proactive, before the fact

This pattern is all about to prevent the system from overloading. This intent is really similar to the one that was mentioned at throttling.

**_Then what is the difference?_**

In case of throttling we observe the load from a _predefined angel_ (for example: user session, IP-address, etc.) On the other hand when we talk about load shedding then we monitor the _overall hit_ of the servers.

**_How does it prevent from the overburden?_**

There is a predefined _load watermark_ under which there is no restriction. This limit is a bit bellow the server’s capacity to protect the headroom for the mission critical processes.  
When the threshold is reached then the entry point can chose if either it rejects the requests blindly or prioritises them and then rejects the lower ones.

As you can see we sacrifice some requests on the altar of availability. It is **still better to have some annoyed clients than having an outage with a lot of pissed off clients**.

You should also remember that because we preserved some extra capacity (because of the lower threshold) we can withstand for short bursts or we can gracefully shutdown.

One final thought that is worth mentioning is the usage of prioritisation. **Preferring high bandwidth clients over clients with slow connection** means that you use your resources wisely.
- You are not occupying your resources to transfer data back to slow clients
- Your throughput can be high because you are not dealing with slow responses
- Preferring quick rejections over slow / no response at all is a good robust strategy

**Application**: The applicability of this pattern is similar to the throttling.

But there is one thing that is worth mentioning. A pretty common pattern is to detach this protection mechanism from the target system. This can be achieved by the following:
- The target system exposes an API endpoint through which the health and capacity can be retrieved periodically
- The first entry point of the ecosystem can be treated / seen as the shock absorber.
    - This means that the load balancers can fulfill this role as well and can provide the load shedding capability.
    - With this in mind the load balancer has a richer / broader meaning.

### Back Pressure

**Categories**: proactive, before the fact

This pattern can be seen as the provider-side part of the retry or circuit breaker patterns (both are used on the consumer-side).

The back pressure is (again) a protection mechanism against overloading. Instead of rejecting blindly (or smartly) the requests rather **it** **informs the consumer that “I’m currently full, please come back later“.**

As you can see this pattern can not be applied for simple clients, because the consumer needs to understand this sort of gentle request and act accordingly.

**_How does it work in case of Web APIs?_**

If the endpoint is exposed via HTTP then the provider can use the `429` (Too Many Requests) or the `503` (Service is unavailable) status code to indicate the request for back off.

**_What is the difference between `429` and `503`?_**

- 503 indicates that a service is unavailable for all of the users for a period of time (a.k.a down-time)
- 429 indicates that a given user/machine instantiated too many requests

**_Which one is the preferred?_**

- Generally speaking the 4xx means there is client error, so the problem can be solved by the consumer by altering the request.
- 5xx means something went wrong on the server side, so there is a fault and it is less likely that a new request from the client can solve the problem.
- By looking the question from this angle the answer is that 429 is the preferred way.

**_How would the client know when should it retry?_**

- In case of HTTP the provider can use the [Retry-After header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After) to specify when the consumer should instantiate its next request.
- This can be either absolute (for example: `Wed, 29 Apr 2020 08:11:00 GMT`) or a relative (for example: `120` seconds).
- Please bear in mind that some providers might use a different header, for example Azure uses `x-ms-retry-after-ms`

**_How does this fit into the retry + circuit breaker pattern combo?_**

If the provider knows that it will not accept further requests during a specified time then this knowledge can be used inside the circuit breaker as well.
- Until that time the circuit breaker will be in open state causing immediate rejection. When that time elapsed then it transits into Half-Open and opens the door for the retry policy.

**Application**: This is actually a tricky one.  
How do you know when will the server be able to serve the consumers’ requests?
- If the server itself is overloaded then this could be only just a guess / an estimation.
- If the consumer consumed its capacity / quota then you are basically rate-limiting. So this back pressure technique can be the side-kick for the throttling as well.

If we are talking about the first case then a lot other questions may arise:
- Should I use absolute or relative datetime?
- Should I provide different datetime for every request to disperse my workload?
- What if I want to use staged back off (1st phase 20%, 2nd phase 30%, 3rd phase 50%) then how can I achieve this?
- etc.

Because of these questions some cloud providers (Amazon and Google) have decided that they will not use Back Pressure (in this form). Rather **they suggest to their clients to use exponential back-off logic with randomized jitter.**

### Timeout

**Categories**: reactive, after the fact

**_Is slow response better than no response?_**

- We might say it depends, but most of the time the answer is that **any response is better than no response at all**. Why, because we can then react on that either by fallback, or by retry.
- Secondly this can prevent cascading failure. So, a crack in one subsystem will not spread across services (the failure is contained inside the service boundary) and will not cause chain reaction (domino effect).

**_So if it is that good that it can prevent cascading failure then what’s wrong with it?_**

- The problem is that we don't do anything with the problem itself. The remote server does not know about the fact the consumer has given up waiting. Without proper monitoring and alerting even the slow responses are not spotted.
- Because the server does not know about the leaving consumers that’s why it continues to serve the requests and using valuable resources for … nothing.

**_What can we do to avoid of wasting resources?_**

- Somehow the server needs to know that it is time to give up that request. There are two commonly used techniques: cancellation, deadline
    - Cancellation:
        - You as a consumer give a signal to the server that you are not interested anymore about the answer.
    - On the server side you can check this signal periodically and whenever you reach a milestone then you can decide what to do.
    - Deadline:
    - You as a consumer give a hard deadline to the server that you are willing to wait for the answer until that point in the future.
    - On the server side at each milestone you are checking how much time is left and then you can decide what to do.

**_When to use which?_**

- The common thing about these two techniques is that you need to pass a token / context all the way down to all related components to be able to act accordingly.
- But there are several differences as well:
    - Cancellation is a reactive pattern where deadline is a proactive one
    - Cancellation is more fragile because it assumes that there is a reliable network channel where the consumer can send the cancellation signal
    - Whereas deadline is usually passed along with the original request so less interaction is required
    - Cancellation is natively supported by .NET where deadline is natively supported by Golang
    - etc.

**Application**: As I said in the beginning of this section this pattern is one of the most well-known and widely adapted pattern.
- It is mainly used for I/O related calls (database access, remote service call, event publication, etc.)
- In case of microservices the suggested way would be to use deadline from the API gateway to the last downstream system

## Not covered but commonly used patterns

- [Caching](0.%20Cache.md)
    - Memoization
- Redundancy
    - Auto-Scaling
    - Elasticity
- Monitoring
    - [Healthchecks](Healthchecks.md)
- Immutability
- [Compensating Transactions](Cross%20Shard%20Transactions.md)

# Reference:

1. ~~[Resilient service design](https://github.com/peter-csala/resilience-service-design/blob/main/resilience.md)~~