---
layout: post
category: blog
date: "2016-02-10 15:43 -0500"
author: Eric
tags: 
  - node.js
  - shell
published: true
title: Walking on cloud 9 behind a firewall
---


Cloud9 is great.  But what if your code is behind a firewall?  You are in luck.  You can download the Cloud9 IDE and run it locally.

```
git clone git://github.com/c9/core.git c9sdk
cd c9sdk
scripts/install-sdk.sh
node server.js
```

But if you want to access it from a different machine (also behind the same firewall), you need a few options:

```
 node  ~/c9sdk/server.js -a someuser:somepass --listen 0.0.0.0 
```

You can also specify the port you want to use, as well as the working directory.  Also wouldn't it be nice to use `forever` to start this so you don't need to stary it up by hand?  Hmmm...

Why don't we create a little start up script to make this easier.  The following defaults to your currently logged in username and current working directory.  If not specified, and available port will be chosen at random in the range 8100-8900.

The `-f` option requires that `forever` be installed, as in `sudo npm install -g forever`

```

#!/bin/bash


LOWERPORT=8100
UPPERPORT=8900
WDIR=$(pwd)
USERNAME=$USER
PORT=0
FOREVER=0

while [[ $# > 0 ]]
  do
  key="$1"

  case $key in
    -u|--username)
    USERNAME="$2"
    shift # past argument
    ;;
    -P|--port)
    PORT="$2"
    shift # past argument
    ;;
    -p|--password)
    PASS="$2"
    shift # past argument
    ;;
    -w|--workingdir)
    WDIR="$2"
    shift # past argument
    ;;
    -f|--FOREVER)
    FOREVER=1
    ;;
    *)
            # unknown option
    ;;
  esac
  shift # past argument or value
done

if [ $PORT -eq 0 ]
  then
    while :
    do
        PORT="`shuf -i $LOWERPORT-$UPPERPORT -n 1`"
        ss -lpn | grep -q ":$PORT " || break
    done
fi

if [ "$USERNAME" = "" ] || [ "$PASS" = "" ]
  then
    echo "Usage: c9 -u username -p password [-P port] [-w /path/to/working/directory]"
  else
    if [ $FOREVER -eq 1 ]

        then
        echo Starting Cloud9 in the background on port $PORT
        forever start ~/c9sdk/server.js -a $USERNAME:$PASS -p $PORT --listen 0.0.0.0 -w $WDIR
    else
        node  ~/c9sdk/server.js -a $USERNAME:$PASS -p $PORT --listen 0.0.0.0 -w $WDIR
    fi
fi
```

Now what would be REALLY cool is to be able to install this globally and have multiple user sessions.  Installing the Cloud9 SDK globally gets dicey due to file permissions.  There is a git project to provide a workspace management UI akin to the officle version of Cloud9 at c9.io (https://github.com/AVGP/cloud9hub), though I am not sure if it works with the latest version of Cloud9 or not.


