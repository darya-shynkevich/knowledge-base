A NoSQL database, often referred to as a “Not Only SQL” database, is a non-relational database system designed to efficiently handle the storage and retrieval of unstructured or semi-structured data.

[NoSQL](https://github.com/donnemartin/system-design-primer#nosql) is a collection of data items represented in a key-value store, document store, wide column store, or a graph database.
#### [Why to choose NoSQL?](Why%20to%20choose%20NoSQL?.md)

Different applications have different requirements, and the best choice of technology for one use case may well be different from the best choice for another use case. It therefore seems likely that in the foreseeable future, relational databases will continue to be used alongside a broad variety of nonrelational datastores—an idea that is sometimes called [Polyglot persistence](../Polyglot%20persistence.md).
#### [MapReduce Querying](../OLAP/MapReduce%20Querying.md)

## Primary categories of NoSQL DB

#### [Document database](Document%20database):

Database stores data in a document-like fashion, often utilizing formats such as JSON or BSON. *==Each document is autonomous and has the potential for a unique structure, facilitating the storage of heterogeneous data.==* Examples of document-based NoSQL databases include MongoDB and Couchbase, both of which provide versatility and suitability for use cases involving diverse data structures.
#### [Key-Value Pair Databases](Key-Value%20Pair%20Databases)

*==Data is stored as key-value pairs, where each key serves as an exclusive identifier linked to a value carrying relevant information.==* Key-value databases excel in straightforward read-and-write operations, with ease of scalability. Redis and Amazon DynamoDB exemplify key-value NoSQL databases, renowned for their efficiency in managing simple data operations and adaptability to increased workloads.
#### [Column Family Databases](Column%20Family%20Databases)

*==These databases organize data into groupings known as column families, which comprise interconnected columns.==* This design targets workloads necessitating intensive write operations and proficient querying using predefined row and column keys. Notable instances include Apache Cassandra and HBase, representative of column family-oriented NoSQL databases engineered to handle distinct types of data handling demands.
#### [Graph-Based Databases](Graph-Based%20Databases)

The graph-based category is tailored to managing and querying *==data with intricate relationships and interconnections, as seen in social networks or recommendation systems.==* Graph databases leverage nodes, edges, and attributes to model and store data, streamlining complex traversal and relational-based queries. Neo4j and Amazon Neptune are well-recognized examples of graph-based NoSQL databases, optimized for handling intricate relationship structures.

# References:

1. [NoSQL Distilled](https://martinfowler.com/books/nosql.html) (book)
2. [NoSQL Databases: An Overview](https://www.thoughtworks.com/insights/blog/nosql-databases-overview)
3. [Introduction to NoSQL databases](https://www.youtube.com/watch?v=xQnIN9bW0og&list=PLMCXHnjXnTnvo6alSjVkgxV-VH6EPyvoX&index=19) (video)
4. [Schemaless Data Structures](https://martinfowler.com/articles/schemaless/#implicit-schema)
5. [Schema-on-Read vs Schema-on-Write](https://www.slideshare.net/awadallah/schemaonread-vs-schemaonwrite)
6. [Log Structured Merge Tree](https://www.cs.umb.edu/~poneil/lsmtree.pdf)
7. [Sorted String Tables and Compaction Strategies](https://github.com/scylladb/scylla/wiki/SSTable-compaction-and-compaction-strategies)
8. [Leveled Compaction Cassandra](https://www.datastax.com/blog/leveled-compaction-apache-cassandra)
9. [Scylla DB Compaction](https://github.com/scylladb/scylla/wiki/SSTable-compaction-and-compaction-strategies)
10. [NOSQL Patterns](http://horicky.blogspot.com/2009/11/nosql-patterns.html)
11. [Остаться в живых. Крупный проект на одной NoSQL](https://www.youtube.com/watch?v=ZLOFOxsDJIY&list=PLH-XmS0lSi_yVB6gNPkgA_ziD70q_8JFC&index=17) (video)
12. [5 Tips For Scaling NoSQL Databases: Don’t Trust Assumptions](http://highscalability.com/blog/2014/9/24/5-tips-for-scaling-nosql-databases-dont-trust-assumptionstes.html)
13. [An Unorthodox Approach to Database Design : The Coming of the Shard](http://highscalability.com/blog/2009/8/6/an-unorthodox-approach-to-database-design-the-coming-of-the.html)
14. [The Anatomy Of Search Technology: Blekko’s NoSQL Database](http://highscalability.com/blog/2012/4/25/the-anatomy-of-search-technology-blekkos-nosql-database.html)
15. [Google Spanner's Most Surprising Revelation: NoSQL is Out and NewSQL is In](http://highscalability.com/blog/2012/9/24/google-spanners-most-surprising-revelation-nosql-is-out-and.html)
16. [Schema Design for Time Series Data in MongoDB](https://www.mongodb.com/blog/post/schema-design-for-time-series-data-in-mongodb)