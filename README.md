# PostgreSQL Commandments

Short axioms about how to use Postgres.

[<img src="https://wiki.postgresql.org/images/a/a4/PostgreSQL_logo.3colors.svg" align="right"  width="100">](https://www.postgresql.org/)

#

Avoid long transactions.

Avoid idle transactions.

Don't disable autovacuum.

Don't use 'fsync=off' in production.

Don't remove anything from $DATADIR.

Don't use 'kill -9' against Postgres processes.

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
