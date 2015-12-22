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

It seems you must start your `WHERE` clause the first field mentioned in your index.  Doing so seems to cause CouchBase to use a primary scan instead of the index.  But after that, you can pick and choose from the remaining fields to further narrow down your search with ANDs. (I have not tried yet, but I would think that OR logic requires separate indices.)

Do your best to avoid using any fields that are not in your index.  Doing so will take a long time (as long as rebuilding the index with the new term). Also, be careful with values for boolean fields.  `AND metadata.is_gold = true` is very fast, where as `AND metadata.is_gold = 'true'` takes a long time.

What if you want to do some type of string transform?  Just do the same transform in your query as in your index statement.  For example:

```
CREATE INDEX `lowercase_pert_desc` ON `LINCS`(lower(`metadata`.`pert_desc`)) USING GSI
```

This index will be used by the following query:

```
SELECT * FROM LINCS WHERE lower(metadata.pert_desc) = "somelowercasestring"
```

You can verify that by running EXPLAIN on the query and you should see that it is using the lowercase_pert_desc index.

Finally, as you are optimizing your indices you will want to update them, which you actually can't.  You must first drop your index and then create a new one with the same name.  Easy as:

```
DROP INDEX LINCS.cell_pert
```


