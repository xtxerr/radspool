### radspool

More of a concept than technically anything advanced, this design favors the use of storing the RADIUS accounting payload as JSON formatted object files in a directory serving as the accounting buffer spool on the RADIUS host. radspool sends the data out of this spool in frequent intervals to the final backend. With this approach accounting data won't get lost when the RADIUS database is non-responsive or broken.

#### Description
The idea behind this script is to store accounting data first on the local
filesystem before being further processed and inserted into a SQL backend.

Will SQL rollback when needed (single transaction is started on each accounting file). If all inserts succeed, the JSON accounting data is deleted from the filesystem. In case of issues, the accounting file count in the spool directory will increment and all of them are going to be processed and tried to be inserted again on the next run.

