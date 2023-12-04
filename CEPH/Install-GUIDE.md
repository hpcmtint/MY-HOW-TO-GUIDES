# How to Deploy a Ceph Storage to Bare Virtual Machines - RisingStack Engineering

https://blog.risingstack.com/ceph-storage-deployment-vm/

*   How to Deploy a Ceph Storage to Bare Virtual Machines - RisingStack Engineering
*   Anatomy of a Ceph cluster
*   Ceph Setup
*   Ceph Storage Deployment
*   Install prerequisites on all machines
*   Prepare the admin node
*   Deploy resources
*   Using the Ceph filesystem
*   Mounting with the kernel
*   Attaching with FUSE
*   Using the RADOS gateway
*   Ceph Storage Monitoring
*   Ceph Deployment: Lessons Learned & Next Up

[Ceph](https://ceph.com/en/) is a freely available storage platform that implements object storage on a single distributed computer cluster and provides interfaces for object-, block- and file-level storage. Ceph aims primarily for completely distributed operation without a single point of failure. Ceph storage manages data replication and is generally quite fault-tolerant. As a result of its design, the system is both self-healing and self-managing.

**Ceph has loads of benefits and great features, but the main drawback is that you have to host and manage it yourself. In this post, we’ll check two different approaches of virtual machine deployment with Ceph.**

Anatomy of a Ceph cluster
-------------------------

Before we dive into the actual deployment process, let’s see what we’ll need to fire up for our own Ceph cluster.

There are three services that form the backbone of the cluster

*   **ceph monitors** (ceph-mon) maintain maps of the cluster state and are also responsible for managing authentication between daemons and clients
*   **managers** (ceph-mgr) are responsible for keeping track of runtime metrics and the current state of the Ceph cluster
*   **object storage daemons** (ceph-osd) store data, handle data replication, recovery, rebalancing, and provide some ceph monitoring information.

Additionally, we can add further parts to the cluster to support different storage solutions

*   **metadata servers** (ceph-mds) store metadata on behalf of the Ceph Filesystem
*   **rados gateway** (ceph-rgw) is an HTTP server for interacting with a Ceph Storage Cluster that provides interfaces compatible with OpenStack Swift and Amazon S3.

There are multiple ways of deploying these services. We’ll check two of them:

*   first, using the `ceph/deploy` tool,
*   then a docker-swarm based vm deployment.

Let’s kick it off!

Ceph Setup
----------

Okay, a disclaimer first. As this is not a production infrastructure, we’ll cut a couple of corners.

**You should not run multiple different Ceph demons on the same host, but for the sake of simplicity, we’ll only use 3 virtual machines for the whole cluster.**

In the case of OSDs, you can run multiple of them on the same host, but using the same storage drive for multiple instances is a bad idea as the disk’s I/O speed might limit the OSD daemons’ performance.

For this tutorial, I’ve created 4 EC2 machines in AWS: 3 for Ceph itself and 1 admin node. For ceph-deploy to work, the admin node requires passwordless SSH access to the nodes and that SSH user has to have passwordless sudo privileges.

In my case, as all machines are in the same subnet on AWS, connectivity between them is not an issue. However, in other cases editing the hosts file might be necessary to ensure proper connection.

Depending on where you deploy Ceph security groups, firewall settings or other resources have to be adjusted to open these ports

*   22 for SSH
*   6789 for monitors
*   6800:7300 for OSDs, managers and metadata servers
*   8080 for dashboard
*   7480 for rados gateway

Without further ado, let’s start deployment.

Ceph Storage Deployment
-----------------------

### Install prerequisites on all machines

$ sudo apt update
$ sudo apt -y install ntp python

For Ceph to work seamlessly, we have to make sure the system clocks are not skewed. The suggested solution is to install ntp on all machines and it will take care of the problem. While we’re at it, let’s install python on all hosts as ceph-deploy depends on it being available on the target machines.

### Prepare the admin node

$ ssh -i ~/.ssh/id\_rsa -A ubuntu@13.53.36.123

As all the machines have my public key added to `known_hosts` thanks to AWS, I can use ssh agent forwarding to access the Ceph machines from the admin node. The first line ensures that my local ssh agent has the proper key in use and the -A flag takes care of forwarding my key.

$ wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
echo deb https://download.ceph.com/debian-nautilus/ $(lsb\_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
$ sudo apt update
$ sudo apt -y install ceph-deploy

We’ll use the latest nautilus release in this example. If you want to deploy a different version, just change the `debian-nautilus` part to your desired release (luminous, mimic, etc.).

$ echo "StrictHostKeyChecking no" | sudo tee -a /etc/ssh/ssh\_config > /dev/null

OR

$ ssh-keyscan -H 10.0.0.124,10.0.0.216,10.0.0.104 >> ~/.ssh/known\_hosts

Ceph-deploy uses SSH connections to manage the nodes we provide. Each time you SSH to a machine that is not in the list of known\_hosts `(~/.ssh/known_hosts)`, you’ll get prompted whether you want to continue connecting or not. This interruption does not mesh well with the deployment process, so we either have to use `ssh-keyscan` to grab the fingerprint of all the target machines or disable the strict host key checking outright.

10.0.0.124 ip\-10\-0\-0\-124.eu-north\-1.compute.internal ip\-10\-0\-0\-124
10.0.0.216 ip\-10\-0\-0\-216.eu-north\-1.compute.internal ip\-10\-0\-0\-216
10.0.0.104 ip\-10\-0\-0\-104.eu-north\-1.compute.internal ip\-10\-0\-0\-104

Even though the target machines are in the same subnet as our admin and they can access each other, we have to add them to the hosts file (/etc/hosts) for ceph-deploy to work properly. Ceph-deploy creates monitors by the provided hostname, so make sure it matches the actual hostname of the machines otherwise the monitors won’t be able to join the quorum and the deployment fails. Don’t forget to reboot the admin node for the changes to take effect.

$ mkdir ceph-deploy
$ cd ceph-deploy

As a final step of the preparation, let’s create a dedicated folder as ceph-deploy will create multiple config and key files during the process.

### Deploy resources

$ ceph-deploy new ip\-10\-0\-0\-124 ip\-10\-0\-0\-216 ip\-10\-0\-0\-104

The command `ceph-deploy new` creates the necessary files for the deployment. Pass it the hostnames of the **monitor** nodes, and it will create `cepf.conf` and `ceph.mon.keyring` along with a log file.

The ceph-conf should look something like this

\[global\]
fsid = 0572e283-306a-49df-a134-4409ac3f11da
mon\_initial\_members = ip-10\-0\-0\-124, ip-10\-0\-0\-216, ip-10\-0\-0\-104
mon\_host = 10.0.0.124,10.0.0.216,10.0.0.104
auth\_cluster\_required = cephx
auth\_service\_required = cephx
auth\_client\_required = cephx

It has a unique ID called `fsid`, the monitor hostnames and addresses and the authentication modes. Ceph provides two authentication modes: none (anyone can access data without authentication) or cephx (key based authentication).

The other file, the monitor keyring is another important piece of the puzzle, as all monitors must have identical keyrings in a cluster with multiple monitors. Luckily ceph-deploy takes care of the propagation of the key file during virtual deployments.

$ ceph-deploy install \--release nautilus ip-10-0-0-124 ip-10-0-0-216 ip-10-0-0-104

As you might have noticed so far, we haven’t installed ceph on the target nodes yet. We could do that one-by-one, but a more convenient way is to let ceph-deploy take care of the task. Don’t forget to specify the release of your choice, otherwise you might run into a mismatch between your admin and targets.

$ ceph\-deploy mon create\-initial

Finally, the first piece of the cluster is up and running! `create-initial` will deploy the monitors specified in `ceph.conf` we generated previously and also gather various key files. The command will only complete successfully if all the monitors are up and in the quorum.

$ ceph-deploy admin ip-10\-0\-0\-124 ip-10\-0\-0\-216 ip-10\-0\-0\-104

Executing ceph-deploy admin will push a Ceph configuration file and the `ceph.client.admin.keyring` to the `/etc/ceph` directory of the nodes, so we can use the ceph CLI without having to provide the ceph.client.admin.keyring each time to execute a command.

At this point, we can take a peek at our cluster. Let’s SSH into a target machine (we can do it directly from the admin node thanks to agent forwarding) and run `sudo ceph status`.

$ sudo ceph status
  cluster:
	id: 	0572e283\-306a-49df-a134-4409ac3f11da
	health: HEALTH\_OK

  services:
	mon: 3 daemons, quorum ip-10-0-0-104,ip-10-0-0-124,ip-10-0-0-216 (age 110m)
mgr: no daemons active
osd: 0 osds: 0 up, 0 in

  data:
  	pools:   0 pools, 0 pgs
objects: 0 objects, 0 B
	usage:   0 B used, 0 B / 0 B avail
pgs:

Here we get a quick overview of what we have so far. Our cluster seems to be healthy and all three monitors are listed under services. Let’s go back to the admin and continue adding pieces.

$ ceph-deploy mgr create ip\-10\-0\-0\-124

For luminous+ builds a manager daemon is required. It’s responsible for monitoring the state of the Cluster and also manages modules/plugins.

Okay, now we have all the management in place, let’s add some storage to the cluster to make it actually useful, shall we?

First, we have to find out (on each target machine) the label of the drive we want to use. To fetch the list of available disks on a specific node, run

$ ceph-deploy disk list ip-10\-0\-0\-104

Here’s a sample output:

![ceph storage deploy sample output](https://blog.risingstack.com/wp-content/uploads/2021/07/ceph-storage-deploy-sample-output.png)

$ ceph-deploy osd create \--data /dev/nvme1n1 ip-10-0-0-124
$ ceph-deploy osd create \--data /dev/nvme1n1 ip-10-0-0-216
$ ceph-deploy osd create \--data /dev/nvme1n1 ip-10-0-0-104

In my case the label was `nvme1n1` on all 3 machines (courtesy of AWS), so to add OSDs to the cluster I just ran these 3 commands.

At this point, our cluster is basically ready. We can run `ceph status` to see that our monitors, managers and OSDs are up and running. But nobody wants to SSH into a machine every time to check the status of the cluster. Luckily there’s a pretty neat dashboard that comes with Ceph, we just have to enable it.

…Or at least that’s what I thought. The dashboard was introduced in luminous release and was further improved in mimic. However, currently we’re deploying nautilus, the latest version of Ceph. After trying the usual way of enabling the dashboard via a manager

$ sudo ceph mgr module enable dashboard

we get an error message saying `Error ENOENT: all mgr daemons do not support module 'dashboard', pass --force to force enablement`.

Turns out, in nautilus the dashboard package is no longer installed by default. We can check the available modules by running

$ sudo ceph mgr module ls

and as expected, dashboard is not there, it comes in a form a separate package. So we have to install it first, luckily it’s pretty easy.

$ sudo apt install -y ceph-mgr-dashboard

Now we can enable it, right? Not so fast. There’s a dependency that has to be installed on all manager hosts, otherwise we get a slightly cryptic error message saying `Error EIO: Module 'dashboard' has experienced an error and cannot handle commands: No module named routes`.

$ sudo apt install -y python-routes

We’re all set to enable the dashboard module now. As it’s a public-facing page that requires login, we should set up a cert for SSL. For the sake of simplicity, I’ve just disabled the SSL feature. You should never do this in production, check out the [official docs](http://docs.ceph.com/docs/nautilus/mgr/dashboard/#configuration) to see how to set up a cert properly. Also, we’ll need to create an admin user so we can log in to our dashboard.

$ sudo ceph mgr module enable dashboard
$ sudo ceph config set mgr mgr/dashboard/ssl false
$ sudo ceph dashboard ac-user-create admin secret administrator

By default, the dashboard is available on the host running the manager on port 8080. After logging in, we get an overview of the cluster status, and under the cluster menu, we get really detailed overviews of each running daemon.

![ceph storage deployment dashboard](https://blog.risingstack.com/wp-content/uploads/2021/07/ceph-storage-deployment-dashboard.png)

![ceph cluster dashboard](https://blog.risingstack.com/wp-content/uploads/2021/07/ceph-cluster-dashboard.png)

If we try to navigate to the `Filesystems` or `Object Gateway` tabs, we get a notification that we haven’t configured the required resources to access these features. Our cluster can only be used as a block storage right now. We have to deploy a couple of extra things to extend its usability.

> Quick detour: In case you’re looking for a company that can help you with Ceph, or DevOps in general, feel free to reach out to us at [RisingStack](https://risingstack.com/)!

Using the Ceph filesystem
-------------------------

Going back to our admin node, running

$ ceph-deploy mds create ip\-10\-0\-0\-124 ip\-10\-0\-0\-216 ip\-10\-0\-0\-104

will create metadata servers, that will be inactive for now, as we haven’t enabled the feature yet. First, we need to create two RADOS pools, one for the actual data and one for the metadata.

$ sudo ceph osd pool create cephfs\_data 8
$ sudo ceph osd pool create cephfs\_metadata 8

There are a couple of things to consider when creating pools that we won’t cover here. Please consult the [documentation](http://docs.ceph.com/docs/master/rados/operations/pools/) for further details.

After creating the required pools, we’re ready to enable the filesystem feature

$ sudo ceph fs new cephfs cephfs\_metadata cephfs\_data

The MDS daemons will now be able to enter an active state, and we are ready to mount the filesystem. We have two options to do that, via the kernel driver or as [FUSE](https://en.wikipedia.org/wiki/Filesystem_in_Userspace) with `ceph-fuse`.

Before we continue with the mounting, let’s create a user keyring that we can use in both solutions for authorization and authentication as we have cephx enabled. There are multiple restrictions that can be set up when creating a new key specified in the [docs](http://docs.ceph.com/docs/nautilus/cephfs/client-auth/#cephfs-client-capabilities). For example:

$ sudo ceph auth get-or-create client.user mon 'allow r' mds 'allow r, allow rw path=/home/cephfs' osd 'allow rw pool=cephfs\_data' -o /etc/ceph/ceph.client.user.keyring

will create a new client key with the name `user` and output it into `ceph.client.user.keyring`. It will provide write access for the MDS only to the `/home/cephfs` directory, and the client will only have write access within the `cephfs_data` pool.

### Mounting with the kernel

Now let’s create a dedicated directory and then use the key from the previously generated keyring to mount the filesystem with the kernel.

$ sudo mkdir /mnt/mycephfs
$ sudo mount -t ceph 13.53.114.94:6789:/ /mnt/mycephfs -o name=user,secret=AQBxnDFdS5atIxAAV0rL9klnSxwy6EFpR/EFbg==

### Attaching with FUSE

Mounting the filesystem with FUSE is not much different either. It requires installing the `ceph-fuse` package.

$ sudo apt install -y ceph-fuse

Before we run the command we have to retrieve the `ceph.conf` and `ceph.client.user.keyring` files from the Ceph host and put the in /etc/ceph. The easiest solution is to use `scp`.

$ sudo scp ubuntu@13.53.114.94:/etc/ceph/ceph.conf /etc/ceph/ceph.conf
$ sudo scp ubuntu@13.53.114.94:/etc/ceph/ceph.client.user.keyring /etc/ceph/ceph.keyring

Now we are ready to mount the filesystem.

$ sudo mkdir cephfs
$ sudo ceph-fuse -m 13.53.114.94:6789 cephfs

Using the RADOS gateway
-----------------------

To enable the S3 management feature of the cluster, we have to add one final piece, the rados gateway.

$ ceph-deploy rgw create ip\-10\-0\-0\-124

For the dashboard, it’s required to create a `radosgw-admin` user with the `system` flag to enable the Object Storage management interface. We also have to provide the user’s `access_key` and `secret_key` to the dashboard before we can start using it.

$ sudo radosgw\-admin user create \--uid=rg\_wadmin --display-name=rgw\_admin --system
$ sudo ceph dashboard set\-rgw\-api\-access\-key <access\_key\>
$ sudo ceph dashboard set\-rgw\-api\-secret\-key <secret\_key\>

Using the Ceph Object Storage is really easy as RGW provides an interface identical to S3. You can use your existing S3 requests and code without any modifications, just have to change the connection string, access, and secret keys.

Ceph Storage Monitoring
-----------------------

The dashboard we’ve deployed shows a lot of useful information about our cluster, but monitoring is not its strongest suit. Luckily Ceph comes with a Prometheus module. After enabling it by running:

$ sudo ceph mgr module enable prometheus

A wide variety of metrics will be available on the given host on port 9283 by default. To make use of these exposed data, we’ll have to set up a prometheus instance.

**I strongly suggest running the following containers on a separate machine from your Ceph cluster. In case you are just experimenting (like me) and don’t want to use a lot of VMs, make sure you have enough memory and CPU left on your virtual machine before firing up docker, as it can lead to strange behaviour and crashes if it runs out of resources.**

There are multiple ways of firing up Prometheus, probably the most convenient is with docker. After installing [docker](https://docs.docker.com/install/) on your machine, create a `prometheus.yml` file to provide the endpoint where it can access our Ceph metrics.

\# /etc/prometheus.yml

scrape\_configs:
  \- job\_name: 'ceph'
    \# metrics\_path defaults to '/metrics'
    \# scheme defaults to 'http'.
    static\_configs:
    \- targets: \['13.53.114.94:9283\]

Then launch the container itself by running:

$ sudo docker run -p 9090:9090 -v /etc/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus

Prometheus will start scraping our data, and it will show up on its dashboard. We can access it on port `9090` on its host machine. Prometheus dashboard is great but does not provide a very eye-pleasing dashboard. That’s the main reason why it’s usually used in pair with Graphana, which provides awesome visualizations for the data provided by Prometheus. It can be launched with docker as well.

$ sudo docker run -d -p 3000:3000 grafana/grafana

Grafana is fantastic when it comes to visualizations, but setting up dashboards can be a daunting task. To make our lives easier, we can load one of the pre-prepared dashboards, for example [this one](https://grafana.com/grafana/dashboards/7056).

![ceph storage grafana monitoring](https://blog.risingstack.com/wp-content/uploads/2021/07/ceph-storage-grafana-monitoring.png)

Ceph Deployment: Lessons Learned & Next Up
------------------------------------------

CEPH can be a great alternative to AWS S3 or other object storages when running in the public operating your service in the private cloud is simply not an option. The fact that it provides an S3 compatible interface makes it a lot easier to port other tools that were written with a “cloud first” mentality. It also plays nicely with Prometheus, thus you don’t need to worry about setting up proper monitoring for it, or you can swap it a more simple, more battle-hardened solution such as Nagios.

In this article, we deployed CEPH to bare virtual machines, but you might need to integrate it into your [Kubernetes](https://blog.risingstack.com/glossary/kubernetes/)Kubernetes (often abbreviated as K8s) offers a framework to run distributed systems efficiently. It's a platform that helps managing containerized workloads and services, and even takes care of scaling. Google open-sourced it in 2014. or Docker Swarm cluster. While it is perfectly fine to install it on VMs next to your container orchestration tool, you might want to leverage the services they provide when you deploy your CEPH cluster. If that is your use case, stay tuned for our next post covering CEPH where we’ll take a look at the black magic required to use CEPH on Docker Swarm and Kubernetes.

**In the next CEPH tutorial which we’ll release next week, we’re going to take a look at valid ceph storage alternatives with Docker or with Kubernetes.**

> PS: Feel free to reach out to us at [RisingStack](https://risingstack.com/devops-sre-cloud-consulting-services) in case you need help with Ceph or Ops in general!
