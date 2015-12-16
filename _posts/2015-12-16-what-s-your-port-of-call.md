---
layout: post
category: blog
date: "2015-12-16 10:54 -0500"
author: Eric
tags: 
  - code snippet
  - node.js
published: true
title: "What's your port of call?"
---


It is often helpful when developing a node app to have different ports for development and deployment.  Here are a few approaches:

### Via multiple config environments.
**/project_root/config.js:**
```
var config = {
	'prod': {
 	    'version': 1.0,
 	    'port': 8080
	},
	'devel': {
 	    'version': 1.0,
 	    'port': 8084
	}
};

if(process.env.LINCS_DEVEL){
	module.exports = config.devel;
} else {
	module.exports = config.prod;	
}
```

**/project_root/bin/app.js**

```
var config = require('../config');
var port = config.port; 
```

And finally, to automatically use devel settings when running tests:
**/project_root/test/test.js**

```
process.env.LINCS_DEVEL = 'true';
// then your tests...
```

Conveniently, the scope of process.env is limited to that process, it does not touch your global (or even current terminal session) env settings.

Via environment variables
If you want further flexibility and want to be other to run your app on some arbitrary port to avoid conflicts with other team members, you can always do `export MYPORT=8881` at the command line and then grab that value in your app (the example below combines this approach with the above config.js approach for maximum flexiblity):

**/project_root/bin/app.js**

```
var config = require('../config');
if(process.env.COUCHLINCS_PORT) {
   port = process.env.COUCHLINCS_PORT;
} else {
   var port = config.port; 
}
```

