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

In case a specific SQL insert for whatever reason fails, the file we be left on
disk. Each additional execution of radspool will parse and try to commit again.
$SPOOLDIR will grow in that case by one new accounting file, each time the
script is run again (the most recent Radiator acctlog-combined.json file will
be moved into the spool directory). In certain situations it might be
beneficial to have the flexibility in eg. configuring the transaction start/end
points. In high volume with large log files small transactions can reduce
storage usage. In other setups it's more desirable to optimize for CPU and IO.
Depending on the individual problem some configuration knobs might be needed to
deliver some flexibility in that regard.

(this code does basically ..
1. move current active JSON accounting file to $SPOOLDIR
2. build array of files from $SPOOLDIR and loop through
3. open next file, parse it's JSON, prepare SQL insert with the new data
4. on any exception in any previous step do a SQL rollback and skip file
5. do a SQL commit and delete the file
)
