If we want to make it easier to write software, so that we can build more powerful systems more cheaply, we must find ways to make software simpler. (…) Most of the code in any system is written by extending the existing code base, so your most important job as a developer is to **facilitate those future extensions**. Thus, you should not think of “working code” as your primary goal, though of course your code must work. Your primary goal must be to produce a great design, which also happens to work. This is strategic programming. (…) If software developers should always be thinking about design issues, and reducing complexity is the most important element of software design, then software developers should always be thinking about complexity. [_A Philosophy of Software Design_](https://web.stanford.edu/~ouster/cgi-bin/aposd.php), John Ousterhout

**Why it is so bad?**
1. **A well-designed system will degrade into a badly designed system over time:** a well-designed system is an unstable, ephemeral state; whereas a badly designed system is a stable, persistent state.
2. **Complexity is a Moat (filled by Leaky Abstractions):** when systems compete with each other for market share, delicacy goes out the window and designers often give the application everything it wants.
3. **There is no fundamental upper limit on Software Complexity:** complexity is limited only by human creativity

**How to?**
- We should **aim for simplicity** because 
	- it is a prerequisite for reliability
	- ease of understanding
	- ease of change
	- ease of debugging
	- flexibility
- **Simple != easy**. "Easy" means "to be at hand", "to be approachable". "Simple" is the opposite of "complex" which means "being intertwined", "being tied together". 
- **What matters** in software is: *does the software do what is supposed to do? Is it of high quality? Can we rely on it? Can problems be fixed along the way? Can requirements change over time?* The answers to these questions is what matters in writing software not the look and feel of the experience writing the code or the cultural implications of it.
- **Build simple systems by**: 
    - Abstracting - design by answering questions related to what, who, when, where, why, and how.
    - Choosing constructs that generate simple artifacts.
    - Simplify by encapsulation.

**Overengineering** can lead to the following problems:
1. **Increased complexity**: The more complex a system is, the harder it is to maintain, test, and debug.
2. **Increased development time**: Adding unnecessary features or functionality will take longer to develop and test.
3. **Increased maintenance costs**: Overengineered systems are harder to maintain, meaning that it will cost more to keep them running.
4. **Reduced performance**: Overengineered systems may perform worse than simpler systems, due to the increased complexity.

# References:

1. https://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.93.8928 TBD
2. https://www.infoq.com/presentations/Simple-Made-Easy/
3. [Why Static Languages Suffer From Complexity](https://hirrolot.github.io/posts/why-static-languages-suffer-from-complexity.html#)
4. [Use Solid Technologies - Don’t Re-invent the Wheel - Keep It Simple!](https://medium.com/@DataStax/instagram-engineerings-3-rules-to-a-scalable-cloud-application-architecture-c44afed31406)
5. [Simplicity by Distributing Complexity](https://jobs.zalando.com/tech/blog/simplicity-by-distributing-complexity/)
6. [Why Over-Reusing is Bad](http://tech.transferwise.com/why-over-reusing-is-bad/)
7. [Avoid Over Engineering](https://medium.com/@rdsubhas/10-modern-software-engineering-mistakes-bc67fbef4fc8)
8. ~~[Three Laws of Software Complexity (or: why software engineers are always grumpy)](https://maheshba.bitbucket.io/blog/2024/05/08/2024-ThreeLaws.html)~~
9. ~~[A Note on Essential Complexity](https://olano.dev/blog/a-note-on-essential-complexity/?utm_source=substack&utm_medium=email)~~