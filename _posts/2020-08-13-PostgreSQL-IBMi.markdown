---
layout: single
title: Connecting PostgreSQL from IBM i
layout: single
categories: ibmi jdbc rpgle free-format 
tags: ibmi as/400 iseries
---
Here I explain how to connect PostgreSQL database from IBM i using type 4 JDBC driver in a RPGLE program. 

Recently, my client acquired an application which uses PostgreSQL database. The main application resides on IBM i. They wanted to get the data from PostgreSQL into IBM i. 

I did check, if they are open to install PostgreSQL on IBM i itself, which may save them additional infrastructure costs, but the "management" was not ready and I had no power/influence on their decision.

Long story short, they asked me to confirm if we can connect in any way.

I started on the task, downloaded [Scott Klement's](https://www.scottklement.com/jdbc/) JDBC driver Service Program. 
Here are the steps I took, 

1. Downloaded type 4 JDBC driver jar from [PostgreSQL](https://jdbc.postgresql.org) server, placed it in home folder in IFS. e.g. `/home/UserID/java/jdbc/POSTGRESQL-42.2.14.jar`
2. Derived their Class.forName string `'org.postgresql.Driver'` from their [documentation](https://jdbc.postgresql.org/documentation/documentation.html). This is needed when connecting to the database, see the source.
3. There are few environment variables which needs to be set for Java to work from RPGLE. 
    * the environment variable - `CLASSPATH` for driver **.jar** file in *IFS*.
    * setting `STDIN`, `STDOUT`, `STDERR` to catch Java exceptions. Also, a call to `CHECKSTDIO` program to set these 3 open. [*more details here*](https://www.ibm.com/developerworks/rational/cafe/docBodyAttachments/2681-102-2-7220/Troubleshooting_RPG_Calls_To_Java_v2.html).
4. `RPGLE` program source is [here](https://github.com/anand-khekale/PostgreSQL-IBMi). 
5. With RPGLE, Java and correct JDBC driver, this works perfectly. 

Thanks to the `JDBCR4` service program provided by Scott Klement, the development time has shortened considerably.
Feel free to reach me on [Twitter](https://twitter.com/anandkhekale). 