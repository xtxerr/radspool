# radspool
radspool.pl brings JSON formatted accounting data from OSC's Radiator RADIUS server into SQL and deals with potential SQL downtimes by spooling all accounting data as long as necessary.

## Description
The idea behind this script is to get accounting data into a file on the local filesystem of the Radiator host. By using JSON formatting the file can then be easily parsed and used by any process and purpose (eg. exporting into ElasticSearch).

This script parses these accounting files and inserts them into a configured SQL backend. If the SQL backend is non-functional and cannot be accessed, the script fails and if necessary will try to SQL rollback, as well as keeping any accounting file not yet inserted into SQL.

If the configured SQL backend is not available eg. not able to be reached, then $SPOOLDIR will grow on each script execution by another accounting file (the active Radiator acctlog-combined.json file will be moved into the spool directory.
