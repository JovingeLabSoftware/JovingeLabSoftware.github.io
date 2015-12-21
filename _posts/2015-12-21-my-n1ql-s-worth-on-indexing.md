---
layout: post
category: blog
date: "2015-12-21 14:42 -0500"
author: Eric
tags: 
  - couchbase
published: true
title: "My n1ql's worth on indexing"
---

Have been doing a lot of indexing this week on a CouchBase document store with about 1.3 million documents. Here is an example of a compound index.  The idea here is to combine all fields that might appear together in a `WHERE` statement with `AND` logic.

```
CREATE INDEX `cell_pert` ON `LINCS`((`metadata`.`cell_id`), (`metadata`.`pert_desc`), (`metadata`.`pert_dose`), (`metadata`.`pert_time`), (`metadata`.`is_gold`)) USING GSI
```

It seems you must start your `WHERE` clause the first field mentioned in your index.  Doing so seems to cause CouchBase to use a primary scan instead of the index.  But after that, you can pick and choose from the remaining fields to further narrow down your search with ANDs.

I have not tried yet, but I would think that OR logic requires separate indices.

Do your best to avoid using any fields that are not in your index.  Doing so will take a long time (as long as rebuilding the index with the new term).

Speaking of rebuilding your index, you must first drop your index prior the creating a new one with the same name (i.e., no "in place" updates).  Easy as:

```
DROP INDEX LINCS.cell_pert
```

