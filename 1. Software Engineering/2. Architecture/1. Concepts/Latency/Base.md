References why it is important: [The Real Cost of Slow Time vs Downtime](The%20Real%20Cost%20of%20Slow%20Time%20vs%20Downtime.md)

Latency is defined as **the time it took one operation to happen.** This means every operation has its own latency—with one million operations there are one million latencies. As a result, latency _cannot_ be measured as _work units / time_. What we’re interested in is how latency _behaves_. To do this meaningfully, we must describe the complete distribution of latencies. Latency almost _never_ follows a normal, Gaussian, or Poisson distribution, so looking at averages, medians, and even standard deviations is useless.

Latency tends to be heavily multi-modal, and part of this is attributed to “hiccups” in response time. Hiccups resemble periodic freezes and can be due to any number of reasons—GC pauses, hypervisor pauses, context switches, interrupts, database reindexing, cache buffer flushes to disk, etc. These hiccups never resemble normal distributions and the shift between modes is often rapid and eclectic.

![Pasted image 20230825104528](../../../../../_Attachments/Pasted%20image%2020230825104528.png)
*With the exception of google.com, every page has a probability of 50% or higher of seeing the 99th percentile.*

Most of the tools that capture the response times, report 99 percentile latency of every 30 sec duration. For example prometheus metrics are scraped every one minute. **==But the real thing to look at is the Max response time.==** 

**Coordinated omission** in software monitoring is when monitoring systems miss or inaccurately measure certain events, leading to distorted performance measurements. For example, if a system experiences frequent small delays, the monitoring system may overlook them, resulting in a misleadingly fast average response time or it's similar to a stopwatch that doesn't account for pauses in a race, resulting in an underestimation of total time. This can conceal problems and mislead developers or analysts in understanding the true performance characteristics of software applications.

`Gatling` fixed the co-ordinated omission problem. Most of the other tools like `Jmeter`, etc still have this problem. So use Gatling for your load generation and reporting purposes. 

When a graph shows sudden spike, it is an indication of a 'possible' coordinated omission. If a graph is smoothly growing it is an indication that there is no bad data. Exceptions maybe there to this rule.

### Measuring Latency

There is no point in looking at percentile graphs if you don't have performance goals set for your service. If you are comparing two systems and your target is 20ms, then you could plot graphs and see what is the maximum throughput each system supports while maintaining latency at 20 ms.

What’s more important is testing the speeds in between idle and hitting the pole. Define your [[Incident response process]]s and plot those requirements, then run different scenarios using different loads and different configurations. This tells us if we’re meeting our SLAs but also how many machines we need to provision to do so. If you don’t do this, you don’t know how many machines you need.

How do we capture this data? In an ideal world, we could store information for _every_ request, but this usually isn’t practical. [HdrHistogram](http://hdrhistogram.org/) is a tool which allows you to capture latency and retain high resolution. It also includes facilities for correcting coordinated omission and plotting latency distributions. The original version of HdrHistogram was written in Java, but there are versions for many other languages.


## References:

1. [Understand Latency](http://highscalability.com/latency-everywhere-and-it-costs-you-sales-how-crush-it)
2. [Everything You Know About Latency Is Wrong](https://bravenewgeek.com/everything-you-know-about-latency-is-wrong/)
3. [Latency Numbers Every Programmer Should Know](http://norvig.com/21-days.html#answers)
4. [Common Bottlenecks](http://highscalability.com/blog/2012/5/16/big-list-of-20-common-bottlenecks.html)
5. [Think Of Latency As A Pseudo-Permanent Network Partition](http://highscalability.com/blog/2010/8/12/think-of-latency-as-a-pseudo-permanent-network-partition.html)
6. [Your Load Generator Is Probably Lying To You - Take The Red Pill And Find Out Why](http://highscalability.com/blog/2015/10/5/your-load-generator-is-probably-lying-to-you-take-the-red-pi.html)
7. [Little’s Law, Scalability And Fault Tolerance: The OS Is Your Bottleneck. What You Can Do?](http://highscalability.com/blog/2014/2/5/littles-law-scalability-and-fault-tolerance-the-os-is-your-b.html)
8. [Optimizing Myntra’s Pricing System for Serving Millions of Traffic under 30ms Latency](https://medium.com/myntra-engineering/optimizing-myntras-pricing-system-for-serving-millions-of-traffic-with-30ms-latency-73a7057affdf)
9. ~~[Everything You Know About Latency Is Wrong](https://bravenewgeek.com/everything-you-know-about-latency-is-wrong/)~~
10. ~~["How NOT to Measure Latency" by Gil Tene](https://www.youtube.com/watch?v=lJ8ydIuPFeU)~~

