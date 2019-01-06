# radspool
radspool.pl brings JSON formatted accounting data from OSC's Radiator RADIUS server into SQL and deals with potential SQL downtimes by spooling all accounting data as long as necessary.

## Description
The idea behind this script is to get accounting data into a file on the local filesystem of the Radiator host. By using JSON formatting the file can then be easily parsed and used by any process and purpose (eg. exporting into ElasticSearch).

This script parses these accounting files and inserts them into a configured SQL backend. If the SQL backend is non-functional, cannot be accessed, has any issue that cause an exception, it will try to SQL rollback (SQL transaction is started on each accounting file) and prevent a commit. Only on successful commit the accounting file which was inserted successfully is deleted.

In case a single SQL insert constructed from an accounting file fail, then, as said, the file we be left on disk until it will be ever committed successfully. $SPOOLDIR grows in that case also by one another accounting file, each time the script is run again (the most recent Radiator acctlog-combined.json file will be moved into the spool directory). If worth do implement one could elaborate if it should be configureable how granular transactions should be be done. In high volume setups a few hours of local disk caching/spooling of accounting files could result in large amounts of data, therefore committing as much data as possible could be beneficial from a storage pov, the overhead of row-level transactions for someone on the otherhand	negligible.
