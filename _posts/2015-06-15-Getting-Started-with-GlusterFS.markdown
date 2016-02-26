---
layout: post
title:  "Getting Started with GlusterFS"
date:   2015-06-19 19:38:21
categories: jekyll update
---

### Motivation
I have recently set out on a journey to become more well versed in open source software.  I've been working a bit with openstack swift, swiftonfile, python-libgfapi and GlusterFS.  I've found that there are a few barriers to entry with some of this software, due to a  combination of a lack of documentation and the the constantly changing nature of opensource software. When I first set out to set up a distributed, replicated GlusterFS cluster I struggled quite a bit.  ***My goal in this post is to help people set up a fully functioning GlusterFS cluster.***

---

### Pre-requisites

- A virtualization environment, I will be using KVM.

- An operating system, I will be using Fedora 21




### What is GlusterFS?

"GlusterFS is a scalable network filesystem suitable for data-intensive tasks such as cloud storage and media streaming. GlusterFS is free and open source software and can utilize common off-the-shelf hardware."

Let's break that down a bit.
- GlusterFS is a network filesystem

A network filesystem (NFS) is a client/server application that allows many  remote or local computers filesystems to be viewed as one large file system.  The user can view, store, and update on a remote computer as if they were on their own computer.

- GlusterFS is scalable

GlusterFS scales linearly.  The speed in which GlusterFS can do reads and writes increases as quickly as new hardware is added.

![GlusterFS Scalability](http://alexcampbell.co/images/gluster_scaling.png)

As we can clearly see, as we increase the amount of virtual servers, the throughput of reads and writes increases linearly.  This is extremely powerful becuase there is no performance bottleneck.

- GlusterFS is suitable for data-intensive tasks

This goes hand in hand with GlusterFS's scalability. ***FINISH ME***

- GlusterFS cab utilize common off-the-shelf hardware.

There is no need to buy a bunch of extremely expensive large disks.  Users can simply set up many pieces of cheap hardware, and GlusterFS will distribute the data for you.

### Who uses it?
Ever heard of Dreamworks?  You know, the people that brought you Shrek and Kung-Fu Panda.  They run on GlusterFS for media streaming and storing entire movies.


Oh what about Pandora, the music people.  They use GlusterFS to stream all of their music directly to you.


How about Pied Piper, the fictional company depicted in the show Silicon Valley.  They use GlusterFS for their badass data compression algorithms:

![GlusterFS on Sillicon Valley](http://alexcampbell.co/images/gluster_sillicon_valley.png)


Alrighty, on to the good stuff...

## Part 1: Creating the Nodes

Make sure you have KVM installed and ready to use.  I would go through this, but that's an entirely different blog post.  I'll trust you can do this.


#### Step 1 - Virtual Machines)
Create 2 brand-spankin' new virtual machines running fedora 21.  We will be clustering them together as our servers, and we will use our host machine (or a third virtual machine) as the client.  I will be using [Virtual Machine Manager](https://virt-manager.org/) and vagrant. Creating virtual machines and using vagrant is not in the scope of this post, **but make sure you know your virtual machine's hostname!** We will need it later.  This can be found with ```hostname``` from inside the terminal


#### Step 2 - Installing GlusterFS)
 We need to install ```glusterfs-server``` on our server machines (our first two virtual machines).  This will allow our client to interact with our servers in the future.

**2a)** Install the GlusterFS server software on your 2 new virtual machines.
```yum install glusterfs-server```

**2b)**  Start the GlusterFS server daemon.
```service glusterd start```

make sure it's running:
``` service glusterd status```


*Optional:*

```chkconfig glusterd on```
This will ensure that glusterd is running everytime we boot.



#### Step 3 - Creating a Trusted Pool)
We want our servers to be able to talk to each other, so we're going to create a trusted pool.  Think of it as a cult: Only current members can recruit new members, and once your in, it is unlikely that you leave.  Welcome to the cult of Gluster.


**3a)**  In order to allow our servers to communicate we're going to need to disable SElinux - which is essentially a kernel level security policy.
```vim /etc/selinux/config```


make sure your config file looks like this:
![selinux](http://alexcampbell.co/images/SElinux.png)

you may need to reboot your virtual machines now.


**3b)**  Okay let's created the trusted cult-pool.  Remember how I said remember your hostname?  You need it now!  If you forgot just type ``hostname``` in your terminal
```gluster peer probe <hostname>```

Note:  Once we probe from say Server1 to Server0 (```gluster peer probe Server0```) both Servers are in our trusted pool.

---


## Part 2: Creating the Bricks and Volumes
Now that we have our trusted pool, we can begin using gluster's bricks and volumes.  A brick is essentially a single filesystem.  A volume is a collection of filesystems that we will mount onto our client.  This will give our client the illusion that we are accessing one filesystem, when in reality we are accessing data from many remote filesytems.  Gluster magic!

#### Step 1 - Create the Bricks)

**1a)** We need to create a new disk device to mount our new brick filesystem on within our virtual machines.
Check out your ```/dev``` folder.  There you should see a few virtual disk devices such as ```/dev/vda``` or ```/dev/vda1```
![vda](http://alexcampbell.co/images/dev_vda.png)


Okay so our goal here is to create another virtual disk device.  Go ahead and add some hardware
![add hardware](http://alexcampbell.co/images/add_hardware.png)


Make sure bus type  is VirtIO
![VirtIO](http://alexcampbell.co/images/virtIO.png)


Once its created check back in your /dev folder.
![dev_vdb](http://alexcampbell.co/images/dev_vdb.png)

Notice the new virtual device named vdb.  This device is completely empty, it doesn't even have a filesystem on it.


**1b)**  Let's make the filesystem.  Run this on *both* virtual machines.
```sudo mkfs.xfs -i size=512 /dev/vdb```

This is saying "make a xfs filesystem with a size of 512 at /dev/vdb".  You should get something like this back:


![mkfs.xfs](http://alexcampbell.co/images/mkfs.png)


**1c)** Create a new folder to mount our new xfs filesystem on.
```mkdir -p /data/brick1 ``` ON VIRTUAL MACHINE 1


``` mkdir -p /data/brick2 ``` ON VIRTUAL MACHINE 2


**1d)**
Edit our /etc/fstab file so that our new filesystem is mounted at /data/brick<number> every time the computer boots.


Add the following to your /etc/fstab file respectively for each virtual machine
```/dev/vdb /data/brick<number> xfs defaults 1 2```

While we're at it, go ahead and mount the filesystem:


``` mount ```


Bam, we just created one brick on two seperate servers for a total of two bricks. Next we'll create a volume out of these two bricks.


#### Step 2 - Creating the Volume)


**2a)** First things first we need to install some damn dependencies... On your third virtual machine (or host machine) run the following command:


```yum install glusterfs-client ```

This includes all the stuff we need to run a client.  Easy enough.


**2b)** We need to update the /etc/hosts file to include the servers on our third virtual machine (or host machine).


Go ahead and open up ```/etc/hosts```.  Find your server's IP addresses by running ```ifconfig``` your ```/etc/hosts``` file should look something like this:


![/etc/hosts](http://alexcampbell.co/images/etchosts)

...With different IPs of course.


**2c)** Okay finally, let's create our volumes.

```gluster volume create volume1 replica 2 virtual1:/data/brick1/volume1/ virtual2:/data/brick2/volume1 ```


Let's break this down a bit.  We're creating a new volume named *volume1* with a *replica value of 2* and using virtual1's brick1 and virtual2's brick2 for our volume.


One of the more important parts here is the replica 2 bit.  In our previous command we created a two node distributed volume with a two-way mirror.  In other words, our data is distributed across 2 nodes, and is replicated twice.


Let's delve deeper into replication.  It is generally thought to have a replication factor of 3.  This accounts for one node going down, and one node being updated or maintained at any one time, thus leaving your data still available.  After 3 replications we start getting diminishing returns.  The likelyhood of needing 4 replicas of a volume is unlikely enough that it is not worth the cost.  That being said, if your life literally depends on the data being there, I'd go with a replication factor of 50.


**2d)** Mounting the volume on the client!

We're almost done.  Let's mount our volume on the client.


```sudo mount -t glusterfs virtual2:/volume1 /mnt/glusterfs/```

Bam we did it.  You should be able to do this now:


<img src="http://alexcampbell.co/images/volume_demo.gif" alt="volume gif" style="width:1000px;height:500pxpx">

---

## Conclusion


Hopefully you got a cluster up and running!  If not, feel free to bug me.  I'm happy to answer any questions people have.  This is the first blog post in a series exploring swift, swiftonfile, and putting it all together into a nodejs application using swiftonfile!
