### radspool

More of a concept than technically anything advanced, this script and approach favors the use of storing RADIUS accounting payload as JSON formatted object files in a directory serving as the accounting buffer spool on the RADIUS host. radspool will send the data out of this spool in frequent intervals (eg. cron) to the final SQL backend. With this approach accounting data won't get lost when the RADIUS database is non-responsive or broken.

#### Description
The idea behind this script is to store accounting data first on the local filesystem before being further processed (eg. replicated into ElasticSearch) and inserted into the SQL backend (Radiator schema). 

When needed the script will rollback any transaction (each JSON file result in single tx) and will keep the file in the spool directory until it's picked up and tried again on the next execution. If all INSERTs from a file succeed the file will be deleted. All files in the spool directoy are tried to be INSERTed on each run. Take care of your database/table/row locking strategy/implementation.
