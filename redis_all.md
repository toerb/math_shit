REDIS
=====

[Redis](http://redis.io/) is the abbreviation for *RE*mote *DI*ctionary *S*erver [1] and is an open-source and cross-plattform member of the key-value databases.
The main goal of Redis is high performance as this was the intention of the inventor Salvatore Sanfilippo.
Back in 2009 he started the Redis project as MySQL was too slow and imperformant for his web analysis tool. [2]

Since 2010 Salvatore Sanfilippo is an employee of VMware [3] and can develop full-time on Redis.
Another key contributor named Pieter Noordhuis was also hired by VMware.
Since then Redis became very popular and has an active community.
According to <http://db-engines.com> Redis is the most common key-value database. [4]

Redis uses its own client protocol which is straightforward to use.
Because of the simple protocol design many language bindings exist for Redis.
Supported programming languages are C, C++, C#, Java, Node.js, Perl, PHP, Python, R, Ruby and many more. [5]

Data types
---

Redis is sometimes described as "Data Structure Server" [6] which implies that it is more than a key-value store.
It does not only use trivial strings as values, but also supports these abstract data types:
* Lists: Store elements sequentially like in an array, access by the index
* Sets: Used when values should be unique, access by the key
* Sorted Sets: Like normal sets but every entry has its score which provides sorting capability
* Hashes: Mapping of N keys to N values, similar to an associative array

For every data type there are special commands available like "SINTER" to get the intersection of multiple sets.
To get all entries of a set there is the command "SMEMBERS". When storing data which is going to expire like session cookies in web applications the command "EXPIRE" can be used. It allows to set an expire date for keys.

If you want to become familiar with the Redis commands, the easiest way is to read the [documentation](http://redis.io/commands) and then test them on [try.redis.io](http://try.redis.io/).

Performance
---

The main reason why Redis performs so extremely well is caused by the in-memory technology.
All data is stored in the main memory instead on slow hard disc drives.
Write and read operations often have the same performance because of that. [7]
Another reason is the key-value design itself.
This allows a simpler implementation and many operations to have the time complexity O(1).

For developers it is also much easier to track performance bottlenecks.
Every available command of Redis is documented with its time complexity in the Big O notation.
Compared with a relational database this makes a huge difference as you never know how the query
gets optimized and executed internally.

While the performance is the key advantage of Redis, it is also the cause for its limitations.
As the whole databases are stored in-memory all the time, the memory consumption is comparatively high.
Nowadays RAM is the most expensive part for servers, especially in the cloud environment (IaaS). [8]
CPU power and hard disc drives are cheap in comparison.

Persistency
---

Another limitation can be persistence but there are two ways to solve that.

The first one is called snapshotting.
It means that every N seconds or rather transactions the data is synchronized on a hard disc drive. [9]
After a restart or a crash of the database, the last written snapshot is read into the memory again.
Snapshotting is the default mode for Redis with a value of 2 seconds.
The alternative to gain persistence is AOF (Append-only File).
Similar to a version control system like Git all write operations are appended to a log file.
By executing all operations sequentially from the AOF the database can be restored.
The tradeoff for guaranteed persistence are of course less performance and a high IO load.

Mutual exclusion
---

In centralised data stores like databases new problems for multithreaded programs arrive. When concurrent access and writes takes place lost updates can occur. A simple example:

    GET x FROM ...
    x++
    UPDATE x

If thread A and thread B with the above code are executed concurrently, there is a possible way for an unwanted result. Let us assume that both A and B are reading x=10. Both will increment x so x=11 now. Now both of them write x back to the database where x is now 11. But the wanted result is two increments of x what should be 12. This is called a lost update.

Redis is now trying to compensate a need for a mutual exclusion in the program accessing the database: The mutual exclusion is in the code for the database interface. The above example would be a one-liner with Redis:

    INCR x

Because of Redis being singlethreaded no lost update can take place.
<br>
Well-known users
---
Various companies and websites use Redis as database because of its high performance and scalability.
Some examples of well-known users are: [10]
* [Twitter](https://twitter.com/)
* [Flickr](https://www.flickr.com/)
* [Tumblr](https://www.tumblr.com/)
* [Instagram](https://instagram.com/)
* [GitHub](https://github.com/)
* [Stack Overflow](https://stackoverflow.com/)
* [Kickstarter](https://www.kickstarter.com/)
* And many more

Comparison: MySQL - Redis
===

MySQL is one of the worldwide most often used relational database management systems
and the most used Open Source Database (DB-Engines, 2015).
So if you want to know how good a database is it does make sense to compare it to MySQL.

The problem is that you can't just compare MySQL and Redis one to one as they are different types of databases.
So we try to look at some areas of the databases where they have advantages and disadvantages.

Data-Selection
---

Key-Value-Databases only support access of data through keywords, so if data should be selected
by values the developer faces the challenge how he still can access these data-sets.
There are different ways how to meet this problem:

* Parse through all data-sets behind a key:
As an example if there is a list of users accessed by an ID and we want to find all users with a specific name,
we need to go through all users and compare the names. If we have many users this means that many operations
have to be executed. It can also get complex quite fast if we have many lists connected with each other.

* Another way would be to accept redundant data. For example if there is a system where users authenticate
by their user-name but internally we user unique IDs for an user we need to map the user-name to the ID.
To find the ID for a user we can use the first way by parsing all users or we have a different list
where we have the user-name as a key and the ID as value.
Through this it's easily possible to map the user-name to the correct ID but we hazard the consequence of redundant data.

Such problems don't occur in relational databases as they are able to also select data by their value.

Injection
---

Security is nowadays a big topic also regarding databases.
The issue of SQL-injection is probably known for every web-developer working with MySQL or other relational databases.
Often in SQL-statements you also save data which was entered by a user.
As an example in social networks data like posts or comments need to be saved in databases.
This opens the possibility for the user to not only insert normal text but also whole SQL-statements which read,
insert, modify or even delete data in the database.

    // expected password: $pwd == "mypassword123"
    // inserted password: $pwd == "hehehe', admin='yes', trusted=100 "
    $query = "UPDATE usertable SET pwd='hehehe', admin='yes', trusted=100 WHERE ...;"[11]

In the example above you can see how you can misuse the password-input for gaining more rights on the database.
Instead of simply typing the password additional data in form of SQL-statements was inserted into the password.
To avoid SQL-injection every user input needs to be escaped so that he can't harm the database which means some extra work.

The problem of injection does not exist in Redis as the Redis protocol doesn't have a concept of string escaping. [1]
This simplifies the code and also helps developers as they don't have to think about such security issues.

Constraints
---

Another difference between MySQL and Redis is the existence of Constraints in MySQL.
Constraints are used to specify rules for data in tables. [12]
If the transaction of data conflicts with a Constraint then the transaction will be aborted.
This functionality ensures the accuracy and reliability of the data in the database. [13]
Through this it is possible to automatically increment IDs or check if a data-set already exists to prevent of unwanted changes.
Redis doesn't provide Constraints so all rules need to be implemented by the developer himself.
This means sometime a lot of extra work and more complex code.
Also the risk is rising that the developer forgets to implement a rule and through this to unwanted change or insert data.


List of references
---

Redis. (March 2015). Redis. Retrieved on 26th March 2015 from redis.io: <http://redis.io/>

DB-Engines. (March 2015). DB-Engines.com. Retrieved on 28th March 2015 from DB-Engines.com: <http://db-engines.com/en/ranking>

PHP Group. (2007). php.net. Retrieved on 30th March 2015 from php.net: <http://php.net/manual/de/security.database.sql-injection.php>

Michael Russo. (October 2010). Redis, from the Ground Up. Retrieved on 28th March 2015 from blog.mjrusso.com:
<http://blog.mjrusso.com/2010/10/17/redis-from-the-ground-up.html>

Salvatore Sanfilippo. (March 2010). VMware: the new Redis home. Retrieved on 28th March 2015 from oldblog.antirez.com: <http://oldblog.antirez.com/post/vmware-the-new-redis-home.html>

Google Code. (March 2015). Benchmarks - Redis. Retrieved on 29th March 2015 from code.google.com:
<https://code.google.com/p/redis/wiki/Benchmarks>

Amazon. (March 2015). AWS | Amazon EC2 | Preise. Retrieved on 30th March 2015 from aws.amazon.com:
<http://aws.amazon.com/de/ec2/pricing/>

Techstacks. (March 2015). Who uses Redis? Retrieved on 30th March 2015 from techstacks.io:
<http://techstacks.io/tech/redis>

Tutorialspoint. (2014). tutorialspoint.com. From tutorialspoint.com: <http://www.tutorialspoint.com/sql/sql-constraints.htm>

w3schools. (30th March 2015). w3schools.com. From w3schools.com: <http://www.w3schools.com/sql/sql_constraints.asp>

Footnotes
---

\[1\] (Redis, 2015) <br>
\[2\] (Michael Russo, 2010) <br>
\[3\] (Salvatore Sanfilippo, 2010) <br>
\[4\] (DB-Engines, 2015) <br>
\[5\] (Redis, 2015) <br>
\[6\] (Michael Russo, 2010) <br>
\[7\] (Google Code, 2015) <br>
\[8\] (Amazon, 2015) <br>
\[9\] (Michael Russo, 2010) <br>
\[10\] (TechStacks, 2015)
\[11\] (PHP Group, 2007) <br>
\[12\] (w3schools, 2015) <br>
\[13\] (Tutorialspoint, 2014)
