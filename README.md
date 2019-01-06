# radspool
Alternative way of getting RADIUS accounting data (from eg.
            Radiator) into a "longterm" storage backend. All a RADIUS server
            needs to support is to write it's accounting data JSON formatted
            onto it's local filesystem. radspool is then responsibility to
            get this data into the final backend (eg. scheduled by cron).
            With the approach of using the local filesystem to write the data
            off first, the accounting data can't be lost that easily anymore
            just due to some higher-layer dependency issues (eg. "SQL
             filtered/down") anymore.

## Description
The idea behind this script is to store accounting data first on the local
filesystem of the Radiator host before it's going to be further processed and
finally put into the SQL/DB backend. By having filesystem level access on the
JSON formatted data, other applications can easily have access and participate
as well (eg. exporting into ElasticSearch).

This script parses these accounting files and inserts them into a configured
SQL backend. If the SQL backend is non-functional, cannot be accessed, has
any certain issue that leads to an exception, it will SQL rollback if needed
(single transaction is started on each accounting file) and a final commit is
omitted. If all inserts succeeded, then the file containing the data for those
inserts is deleted at the end as well.

In case of issue, the accounting file containing the data will not be deleted,
instead it will remain where it is and radspool on it's next execution
(eg. cron) will try to insert the contained data again. $SPOOLDIR will grow
in that case by one new accounting file - each time the script is run, the
most recent and active 'acctlog-combined.json' file will be moved into the
spool directory. In certain situations it might be beneficial to have the
flexibility to configure the scope of the transaction. In high volume setups
with large log files, small transactions could reduce storage usage in
certain situations by being able to commit inserts not affected by the failed
operation. In other setups it's probably much more likely that you want to
optimize for CPU and IO. Depending on the individual problem, some
configuration knobs would be useful to have.
