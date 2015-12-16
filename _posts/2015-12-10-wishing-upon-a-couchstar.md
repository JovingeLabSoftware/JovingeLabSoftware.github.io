---
layout: post
category: blog
date: "2015-12-10 11:58 -0500"
author: Eric
tags: 
  - couchbase
  - shell
  - Cluster
published: true
title: Wishing upon a couchstar
---





For our next trick, we will put together everything we have learned in prior posts and create a starcluster based couchbase server backed by EFS.  (Is EFS a good choice?  Arguments for not using shared drive with couchbase: failover, network latency.  Arguments for using EFS: it is backed up for me so I do not need redundant drives, Amazon says it is fast.  We'll see).

1. Create an new EC2 instance based on a starcluster AMI.  I went with `ami-17331927`. 
2. SSH to that box, and do:

```
sudo apt-get update
sudo apt-get upgrade
wget http://packages.couchbase.com/releases/4.1.0-dp/couchbase-server_4.1.0-dp-ubuntu14.04_amd64.deb
dpkg -i couchbase-server_4.1.0-dp-ubuntu14.04_amd64.deb
```
And we also need to add an entry into /etc/fstab for our EFS volume (adjust fs id as needed):

```
us-west-2c.fs-58dd34f1.efs.us-west-2.amazonaws.com:/ /data nfs defaults 0 0
```


Do NOT setup/initialize the server now, or it will be more challenging to set the data directories on your nodes later. You can also install R if you want, unless you plan on doing your actual number crunching elsewhere.

Then go to the EC2 console at aws.amazon.com and create an image of that instance.

2. Setup the starcluster tools on whatever box you are going to use to launch/manage the cluster.  The latest version is required to support the latest and greatest instance types.  If you are going to manage from a linux box, this will suffice:

```
git clone https://github.com/jtriley/StarCluster.git
cd 	StarCluster
sudo python setup.py install
```

3. Create an IAM user whose security credential we will use to create instances.  Perhaps you could name the user `starcluster`.  Be sure to grab the credential keys for that user when given the opportunity.

4. Create an IAM group (to make it easy to add or remove users in the future) with 	
the `AmazonEC2FullAccess` policy applied (TODO: we could probably be more fine grained that that). You could also name this group 'starcluster' if you wanted.

5. Add your new IAM user to your new IAM group.

6. If you haven't already, generate the OpenSSH formatted private key from a keypair associated with your EC2 account.  This should be the same one you used to create the AMI above.  I use puttygen to convert the key to OpenSSH format.

7. Now update ~/.starcluster/config with your new user's AWS info and your key info, and adjust the AMI id (to match the image you created in step 2 above). The relevant portions of mine looks like this (personal details changed, and also note the USERDATA_SCRIPTS option, which we need later)

```
AWS_ACCESS_KEY_ID = AF89SFP3DFDFSHJHF4A
AWS_SECRET_ACCESS_KEY = KdsfJDfUfdfdsFDA/e3Q8gffdgAWEFWESD9dtgVW
AWS_USER_ID= 999999999999

AWS_REGION_NAME = us-west-2
AWS_REGION_HOST = ec2.us-west-2.amazonaws.com

[key mykey]
KEY_LOCATION=~/.ssh/mykey.rsa

[cluster smallcluster]
# change this to the name of one of the keypair sections defined above
KEYNAME = jlab_star
# number of ec2 instances to launch
CLUSTER_SIZE = 8
CLUSTER_USER = gseadmin
NODE_IMAGE_ID = ami-d2170bb3
NODE_INSTANCE_TYPE = m4.xlarge
# we'll use this script later...
USERDATA_SCRIPTS = /path/to/startup.sh 
PERMISSIONS = ssh, http
SPOT_BID = 0.13
```
https://aws.amazon.com/ec2/spot/bid-advisor/ suggests that at current condition, we will experience a "low" rate of being outbid (and, therefore, our servers disappearing) at that price point.  We will see how that goes. Alternatives are to have the master node an on demand (or provisioned) instances and then add nodes of a couple of different instance types to minimize the probability of a data compromising collapse of the cluster.  Or, obviously, the whole cluster could be on demand or provisioned instances.

We will also open some ports for couchbase

```
[permission couchbase_rest]
IP_PROTOCOL = tcp
FROM_PORT = 8091
TO_PORT = 8091

[permission couchbase_api]
IP_PROTOCOL = tcp
FROM_PORT = 8092
TO_PORT = 8092


```

8. Add the cluster nodes to the security group to which EFS belongs.  To mount the EFS, you must be in the security group to which the EFS volume is assigned.  StarCluster creates a new security group when the cluster is spawned, and destroys it when it is terminated.  So we need to add the EFS security group to each node at boot.  This is accomplished with the following bash script (save this to `/path/to/startup.sh` as noted in USERDATA_SCRIPTS variable in config in step 7 above, and `chmod 655` for good measure):  (**OF COURSE**  I changed personal details below).

```
#!/bin/bash

# tweak these add needed
export AWS_ACCESS_KEY_ID=AF89SFP3DFDFSHJHF4A
export AWS_SECRET_ACCESS_KEY=KdsfJDfUfdfdsFDA/e3Q8gffdgAWEFWESD9dtgVW
export AWS_USER_ID=99999999999999
export AWS_DEFAULT_REGION=us-west-2

# install aws cli tools if missing
hash aws &> /dev/null
if [ $? -eq 1 ]; then
  sudo pip install awscli
fi

# name of the security group to add this instance to
ADD_GROUP=efs_group  # or whatever your efs authorized group is

# lookup instance details
INSTANCE_ID=$(ec2metadata --instance-id)

GROUP_NAME=$(ec2metadata --security-groups)

GROUP_ID=$(aws ec2 describe-security-groups --group-names ${GROUP_NAME} \
           --query 'SecurityGroups[*].{Name:GroupId}' --output text)

ADD_GROUP_ID=$(aws ec2 describe-security-groups --group-names ${ADD_GROUP} \
           --query 'SecurityGroups[*].{Name:GroupId}' --output text)


# add this instance to group
aws ec2 modify-instance-attribute --instance-id ${INSTANCE_ID} --groups ${GROUP_ID} ${ADD_GROUP_ID}


# and now we can mount EFS

mount /data

```

9. And.....go!

```
starcluster start couchstar
```

10. Now we need to initialize couchbase.  I am working on a scripted solution, but in the mean time you can point your browser to 999.99.999.99:8091 (modify with the IP address of the master node) to init the master node of couchcluster, and then do the same for each node (adding each one to the master rather than starting them new servers). 

11. Almost works...the last step is to open port 11211 in addition to 8091 and 8092, as this is required for client interface (oddly enough, since one would think the REST, API, and N1QL ports would be enough). To summarize, you need ports 8091 (Web Admin), 8092 (REST API), 8093 (N1QL), 4369 (Erlang Port Mapper), and 11211 ("Client Interface") open to interact with your CouchBase cluster from another machine.
