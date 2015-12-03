---
layout: post
title:  "How to break your server in one easy step"
date:   2015-12-03
tags:
- couchbase
- shell
---

How many times will chown -R bite me in the butt before I learn?

So had some intermittent problems with my node couchbase related unit tests failing intermittently with 
`Error: open_db_failed: {not_found,no_db_file}`.  Tried restarting the couchbase server and it would not even 
restart.  Did a `tail /opt/couchbase/var/lib/couchbase/logs/couchdb.log`

And got:

```
[couchdb:info,2015-12-03T13:23:00.017Z,couchdb_ns_1@127.0.0.1:<0.190.0>:couch_log:info:41]eacces error opening file "/mnt/lincs/couchbase/data/_users.couch.1" waiting 1000ms to retry
[couchdb:info,2015-12-03T13:23:01.018Z,couchdb_ns_1@127.0.0.1:<0.190.0>:couch_log:info:41]eacces error opening file "/mnt/lincs/couchbase/data/_users.couch.1" waiting 1000ms to retry
...
```

Oops.  Earlier in the day I had done a chown -R to fix some permissions on files in one of our development directories that I had inadvertently `git clone`'d as root.  Obviously did that 
one directory too high and changed the owner of the couchbase server files.

A quick `chown -R couchbase couchbase/*; chgrp -R couchbase couchbase/*; service couchbase start;` fixed the permissions, allowed the 
server to start, and now my node couchbase tests were passing again.

Lesson learned.  Until a few days from now when I do it again.


