# radspool
radspool.pl brings JSON formatted accounting data from OSC's Radiator RADIUS 
server into SQL and deals with potential SQL downtimes by spooling all 
accounting data as long as necessary.

## Description
The idea behind this script is to store accounting data first on the local
filesystem of the Radiator host before it's going to be further processed and
finally put into the SQL/DB backend. By having filesystem level access on the
JSON formatted data, other applications can easily have access and participate
as well (eg. exporting into ElasticSearch)

This script parses these accounting files and inserts them into a configured
SQL backend. If the SQL backend is non-functional, cannot be accessed, has any
certain issue that leads to an exception, it will SQL rollback (single
transaction is started on each accounting file) and a commit is omitted. If a
file was completely inserted into SQL, then it's also deleted afterwards.

In case of any issue, just 1 out of 10000s of inserts failing, the accounting
file containing the data will not be deleted afterwards - it will remain where
it is and radspool will on it's next execution (eg. cron) try to insert the
contained data again.
$SPOOLDIR will grow in that case by one new accounting file - each time the
script is run the most recent active Radiator acctlog-combined.json file will
be moved into the spool directory. In certain situations it might be beneficial
to have the flexibility to configure the scope of the transaction. 
In high volume setups with large log files, small transactions could reduce
storage usage in certain situations by being able to commit more data. In other
setups it's probably much more likely that you want to optimize for CPU and IO.
Depending on the individual problem, some configuration knobs would be useful
to have.


(this code does basically ..
1. move current active JSON accounting file to $SPOOLDIR
2. build array of files from $SPOOLDIR and loop through
3. open next file, parse it's JSON, prepare SQL insert with the new data
4. on any exception in any previous step do a SQL rollback and skip file
5. do a SQL commit and delete the file
)
