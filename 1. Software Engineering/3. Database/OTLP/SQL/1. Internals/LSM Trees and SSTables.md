> could be a great solution for storage + writing, but not for reading

Some databases like Cassandra and RocksDB ***try an approach that indexes more asynchronously***. Using LSM (Log Structured Merge) Trees and SSTables (Serialized String Tables), rather than B-trees, these databases can achieve higher write throughput by ***writing to an in-memory data structure before "flushing" to disk.*** However, this approach often comes at the cost of slower read speeds.

(-) duplicate key stored in SSTable, merging caused space amplification (both read and write)

Most relational databases do not offer an LSM tree storage option. MySQL is an exception: Facebook developed a storage engine for MySQL backed by RocksDB that uses LSM trees, called MyRocks. Percona Server for MySQL also ships with their flavour of this engine, Percona MyRocks. ***If your Django application is write-heavy and you use MySQL, MyRocks is worth checking out***!

# References:

1. ! [Log-Structured Merge Tree](https://itnext.io/log-structured-merge-tree-a79241c959e3)
2. ! [How does the LSM tree work](https://iorilan.medium.com/how-does-the-lsm-tree-work-7a9fa4b54c36)
3. ! [Revisiting B+-tree vs. LSM-tree](https://www.usenix.org/publications/loginonline/revisit-b-tree-vs-lsm-tree-upon-arrival-modern-storage-hardware-built)
4. [On Disk IO, Part 1: Flavors of IO](!https://medium.com/@ifesdjeen/on-disk-io-part-1-flavours-of-io-8e1ace1de017)
5. [Percona MyRocks Introduction](!https://docs.percona.com/percona-server/5.7/myrocks/index.html)
6. [Deep dive into LSM tree](https://vishalrana9915.medium.com/maximising-performance-with-lsm-trees-a-guide-to-dynamic-data-storage-2c904954215a)
7. ~~[Sorry, Amazon DynamoDB is not column-based, and it is not OLAP](https://iorilan.medium.com/interview-sorry-amazon-dynamodb-is-not-column-based-and-it-is-not-olap-ad8f8bd62098)~~