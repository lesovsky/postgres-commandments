# PostgreSQL Commandments

[<img src="https://wiki.postgresql.org/images/a/a4/PostgreSQL_logo.3colors.svg" align="right"  width="100">](https://www.postgresql.org/)

Short axioms about how to use Postgres.

<details>
  <summary>Disclaimer</summary>
  
  - These are not recommendations from the official PostgreSQL community.
  
  - They are my personal observations based on my own experience as PostgreSQL DBA for the past 10 years.
  
  - You can follow or ignore them but the main course of these commandments - prioritise data safety over performance.
</details>

#

Avoid long transactions.

Avoid idle transactions.

Don't disable autovacuum.

Don't use 'fsync=off' in production.

Don't remove anything from $DATADIR.

<details>
  <summary>Don't use 'kill -9' against Postgres processes.</summary>

PostgreSQL’s official documentation states:
It is best not to use SIGKILL to shut down the server. Doing so will prevent the server from releasing shared memory and semaphores, which might then have to be done manually before a new server can be started. (quote end)
Moreover, using SIGKILL against even a single Postgres backend forces to immediately terminate all other backends, re-initialize internal structures and run recovery from last check point, at which database cluster is not available for clients and applications until recovery ends.

In the example below, you can see how Postgres handles SIGKILL:
1) Process with PID 9774 is terminated by SIGKILL.
2) Postgres terminates the rest of its processes (20 processes in total).
3) Reinitializes and runs automatic recovery process (which may take a while in various scenarios).
4) Finishes recovery and starts accepting connections. (edited)

```
1549 @ from  [] LOG:  server process (PID 9774) was terminated by signal 9: Killed
1549 @ from  [] DETAIL:  Failed process was running: SELECT abalance FROM pgbench_accounts WHERE aid = 729760;
1549 @ from  [] LOG:  terminating any other active server processes
9773 postgres@pgbench from [local] [idle] WARNING:  terminating connection because of crash of another server process
9773 postgres@pgbench from [local] [idle] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9773 postgres@pgbench from [local] [idle] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
1816 @ from  [] WARNING:  terminating connection because of crash of another server process
1816 @ from  [] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
1816 @ from  [] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9768 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9768 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9768 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9782 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9782 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9782 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9764 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9764 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9764 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9770 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9770 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9770 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9769 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9769 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9769 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9772 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9772 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9772 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9779 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9779 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9779 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9780 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9780 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9780 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9775 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9775 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9775 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9776 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9776 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9776 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9771 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9771 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9771 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9766 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9766 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9766 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9765 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9765 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9765 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9781 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9781 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9781 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9777 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9777 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9777 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9767 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9767 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9767 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9778 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9778 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9778 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
9783 postgres@pgbench from [local] [SELECT] WARNING:  terminating connection because of crash of another server process
9783 postgres@pgbench from [local] [SELECT] DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
9783 postgres@pgbench from [local] [SELECT] HINT:  In a moment you should be able to reconnect to the database and repeat your command.
1549 @ from  [] LOG:  all server processes terminated; reinitializing
9817 @ from  [] LOG:  database system was interrupted; last known up at 2018-12-03 15:01:23 +05
9817 @ from  [] LOG:  database system was not properly shut down; automatic recovery in progress
9817 @ from  [] LOG:  redo starts at 8/F3C72E70
9817 @ from  [] LOG:  invalid record length at 8/F3C7A390: wanted 24, got 0
9817 @ from  [] LOG:  redo done at 8/F3C7A358
9817 @ from  [] LOG:  last completed transaction was at log time 2018-12-03 21:39:14.667678+05
9817 @ from  [] LOG:  checkpoint starting: end-of-recovery immediate
9817 @ from  [] LOG:  checkpoint complete: wrote 7 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.000 s, sync=0.016 s, total=0.042 s; sync files=7, longest=0.008 s, average=0.002 s; distance=29 kB, estimate=29 kB
1549 @ from  [] LOG:  database system is ready to accept connections
```
</details>


Don't delete rows by billions at a time.

Avoid creating unnecessary indexes.

The  more indexes table has, the slower you can write into it.

Don't use 'listen_addresses = *' on hosts with public access.

Use connection pooler when there are too many connections.

Postgres doesn't 'hang' at shutdown... It has *checkpoint*.

Don't use minor releases that are less than 3, in production.

VACUUM FULL completely blocks access to the table.

Use CONCURRENTLY as much as possible.

The bigger the table, the slower the count(*).

Hot standby != backup.

Don't trust non-validated backup.

Use partitioning for archived data.

Upstream and master candidate should be the same.

Don't re-invent queues, use PgQ.

Keep logs outside of $DATADIR.

#

:elephant: Contributions welcome. Add commandments or ideas through [pull requests](https://github.com/lesovsky/postgres-commandments/pulls) or create an [issue](https://github.com/lesovsky/postgres-commandments/issues) to start a discussion.
