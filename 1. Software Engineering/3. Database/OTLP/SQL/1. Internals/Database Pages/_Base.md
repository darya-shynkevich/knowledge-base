Databases often use fixed-size pages to store data. Tables, collections, rows, columns, indexes, sequences, documents and more eventually end up as bytes in a page. This way the storage engine can be separated from the database frontend responsible for data format and API. Moreover, this makes it easier to read, write or cache data when everything is a page.

Here is an example of SQL Server page layout.

![Pasted image 20231014192657](../../../../../../_Attachments/Pasted%20image%2020231014192657.png)

# A Pool of Pages

Databases read and write in pages. 

When you read a row from a table, 
	1. the database finds the page where the row lives 
	2. the database identifies the file and offset where the page is located on disk. 
	3. the database then asks the OS to read from the file on the particular offset for the length of the page. 
	4. the OS checks its filesystem cache and if the required data isn’t there, the OS issues the read and pulls the page in memory for the database to consume.

The database allocates a pool of memory, often called ***shared or buffer pool***. Pages read from disk are placed in the buffer pool. Once a page is in the buffer pool, not only we get access to the requested row but also other rows in the page too depending on how wide the rows are. This makes reads efficient especially those resulting from index range scans. ***The smaller the rows, the more rows fit in a single page, the more bang for our buck a single I/O gives us.***

The same goes for writes, when a user updates a row, the database finds the page where the row lives, pull the page in the buffer pool and update the row in memory and make a journal entry of the change (often called WAL) persisted to disk. The page can remain in memory so it may receive more writes before it is finally flushed back to disk, minimizing the number of I/Os. 

Deletes and inserts work the same but implementation may vary.

# **Page Content**

==Row-store databases== write rows and all their attributes one after the other packed in the page so that OLTP workloads are better especially write workload.

==Column-store databases== write the rows in pages column by column such OLAP workloads that run a summary fewer fields are more efficient. A single page read will be packed with values from one column, making aggregate functions like SUM much more effective.

==Document based databases== compress documents and store them in page just like row stores and ==graph based databases== persist the connectivity in pages such that page read is efficient for traversing graphs, this also can be tuned for depth vs breadth vs search.

Whether you are storing rows, columns, documents or graphs, the goal is to pack your items in the page such that a page read is effective. The page should give you as much useful information as possible to help with client side workload. If you find yourself reading many pages to do tiny little work consider rethinking your data modeling.

# Small vs Large Pages

Small pages are faster to read and write especially if the page size is closer to the media block size, however the overhead cost of the page header metadata compare to useful data can get high. On the other hand, larger sizes can minimize metadata overhead and page splits but at a cost of higher cold read and write.

Of course this gets very complicated the closer you get to the disk/SSD. Great minds in storage industry are working on technologies like `Zoned` and key value store namespaces in `NVMe` to optimize read/writes between host and media.

Postgres Default page size is [8KB](https://www.postgresql.org/docs/current/storage-page-layout.html), MySQL InnoDB is [16KB](https://dev.mysql.com/doc/refman/5.6/en/innodb-file-space.html), MongoDB WiredTiger is [32KB](https://source.wiredtiger.com/2.5.2/tune_page_sizes.html), SQL Server is [8KB](https://learn.microsoft.com/en-us/sql/relational-databases/pages-and-extents-architecture-guide?view=sql-server-ver16) and Oracle is also [8KB](https://docs.oracle.com/cd/E29127_01/doc.111170/e28967/nsslapd-db-page-size-5dsconf.htm). Database defaults work for most cases but it is important to know these default and be prepared to configure it for your use case.

# How page are stored on Disk

There are many ways we can store and retrieve pages to and from disk. One way is to make a file per table or collection as an array of fixed-size pages. Page 0 followed by page 1 followed by page 2. To read something from disk we need to information, the file name, offset and the length, with this design we have all three!

To read page x, we know the file name from the table, to get the offset it is X *Page_Size, and the length in bytes are the page size.

Example reading table test, assume a page size of 8KB, to read pages 2 through 10, we read the file where table test live, with an offset 16484 (2*8192) for 65536 bytes ((10–2)*8192).

But that is just one way, the beauty of databases is every database implementation is differnet.

# Postgres Page Layout

In Postgres the default page size is 8KB, and here is how it looks like.

![Pasted image 20231014193610](../../../../../../_Attachments/Pasted%20image%2020231014193610.png)

## Page header — 24 bytes

The page must have metadata to describe what is in the page including the free space available. This is a 24 bytes fixed header.

## ItemIds — 4 byte each

This is an array of item pointers (not the items or tuples themselves. Each itemId is a 4 byte offset:length pointer which points to the offset in the page of where the item is and how large is it.

It is the fact that this pointer exist allows the HOT optimization (Heap only tuple), when an update happens to a row in postgres, a new tuple is generated, if the tuple happened to fit in the same page as the old tuple, the HOT optiimization changes the old item id pointer to point to the new tuple. This way indexes and other data structures can still point to the old tuple id. Very powerful.

Although one criticism is the size the item pointers take, at 4 bytes each, if I can store 1000 items, half the page (4KB) is wasted on headers.

> I use items, tuples and rows but there a difference, the row is what the user see, the tuple is the physical instance of the row in the page, the item is the tuple. The same row can have 10 tuples, one active tuple and 7 left for older transactions (MVCC reasons) to read and 2 dead tuples that no one needs any more.

## Items — variable length

This is where the items themselves live in the page one after the other.

## Special — variable length

This section is only applicable to B+Tree [_Base](../../2.%20Indexes/_Base.md) leaf pages where each page links to the previous and forward. Information about page pointers are stored here.

Here is an example of how tuples are referenced.

![Pasted image 20231014194051](../../../../../../_Attachments/Pasted%20image%2020231014194051.png)

# References:

1. ~~[Why is the deleted record still there?](https://iorilan.medium.com/my-friend-asked-these-questions-to-his-interviewers-but-couldnt-get-an-answer-4e79835c14de)~~
2. [How Do Databases Store Tables on Disk? Explained both SSD & HDD](https://www.youtube.com/watch?v=DbxddGtHl70&list=PLQnljOFTspQXjD0HOzN7P2tgzu7scWpl2&index=38) (videos)
3. [Database page splits | The Backend Engineering Show](https://www.youtube.com/watch?v=fnR215jy-X8&list=PLQnljOFTspQUybacGRk1b_p13dgI-SmcZ&index=53) (video)
4. [How databases store data on disk?](https://www.youtube.com/watch?v=haz2h7_xFDk) (video)
5. [The physical structure of InnoDB index pages](https://blog.jcole.us/2013/01/07/the-physical-structure-of-innodb-index-pages/)