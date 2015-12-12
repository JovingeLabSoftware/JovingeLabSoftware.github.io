---
layout: post
category: blog
date: "2015-12-12 10:19 -0500"
author: JLab Skunkworks
tags: 
  - couchbase
  - shell
published: true
title: Pushing view to couchbase
---

Given a view document `myview.ddoc` like this:

```
{
"views": {
"doc_type": {
"map": "function (doc, meta) {\n  emit(doc.type, null)\n}",
"reduce": "_count"
},
"doc_method": {
"map": "function (doc, meta) {\n  if(doc.method) {    \n\t  emit(doc.method, null)\n  }\n}",
"reduce": "_count"
}
}
}

```

This can be deployed with a script like this:

```
#!/bin/bash

COUCH_HOST='58.35.41.999'
BUCKET='MYBUCKET'

curl -X PUT \
	-H 'Content-Type: application/json' \
	http://${COUCH_HOST}:8092/${BUCKET}/_design/debug \
	-d @myview.ddoc
	
curl -X PUT \
	-H 'Content-Type: application/json' \
	http://${COUCH_HOST}:8092/${BUCKET}/_design/dev_debug \
	-d @myview.ddoc
```

