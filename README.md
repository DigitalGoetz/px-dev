![logo](http://i.imgur.com/l8JRhxg.jpg)

[![Docker Pulls](https://img.shields.io/docker/pulls/portworx/px-dev.svg)](https://hub.docker.com/r/portworx/px-dev)
# PX-Developer 1.0

PX-Developer (PX-Dev) is scale-out storage and data services for containers.  PX-Dev itself is deployed as a container with your application stack.  By running PX-Dev with your application stack, you get container-granular controls for storage persistence, capacity management, performance, and availability in a scaleout environment. Deploying the PX-Developer container on a server with Docker Engine turns that server into a scale-out storage node. Storage runs converged with compute and gives bare-metal drive performance. 

PX-Dev offers container granular services such as:

1. Data persistence in a multi node environment
2. Synchronous data availability across multiple availability zones and automatic AZ detection
3. Automatic tiering and class of service enforcement
4. Bring-your-own-key Encryption
5. Shared namespaces across containers running on different servers
6. S3 interfaces and backup to S3
7. Integration with schedulers to automate container placement

Please visit [our offical docs site for more information](http://docs.portworx.com)

>**Note:**<br/>This section below is now deprecated... The full Portworx Documentation site is now [online here](https://portworx.github.io/px-docs) 

# PX-Developer Documentation

PX-Developer aims to improve the storage experience for developer and DevOps teams using containers. PX-Developer is a limited version of the our flagship software PX-Enterprise for production container deployments and we want to develop this solution with the community, including for enterprises. (More on our [blog](http://portworx.com/px-dev-beta/).) [Contact us](https://github.com/portworx/px-dev#contact-us) to share your feedback, work with us, and to request features. Stay tuned for updates on PX-Developer (PX-Dev for short) and our PX-Enterprise release. If you are looking for more information about PX-Enterprise for your production workloads, please follow this link to get started with PX-Enterprise. [Portworx](http://portworx.com/contact-us)

## Install and Quick Start Guides
As you develop and deploy your apps in containers, use PX-Dev for elastic storage capacity, managed performance, and high availability.

 * See our quick start guides on :
  * [Launching PX-Dev with Docker Compose](https://github.com/portworx/px-dev/blob/master/quick-start/README.md)  
  * [Running PX-Dev with Kubernetes](https://github.com/portworx/px-dev/blob/master/install_with_k8s.md)
  * Running PX-Dev on [Ubuntu](https://github.com/portworx/px-dev/blob/master/install_run_ubuntu.md),  [Red Hat](https://github.com/portworx/px-dev/blob/master/install_run_rhel.md), and [ CoreOS ] (https://github.com/portworx/px-dev/blob/master/install_run_coreos.md)
 * Run stateful containers with Docker volumes:
  * [Scaling a Cassandra database with PX-Dev](https://github.com/portworx/px-dev/blob/master/examples/cassandra.md) 
  * [Running the Docker registry with high availability](https://github.com/portworx/px-dev/blob/master/examples/registry.md) 
  * [Running PostgreSQL from CrunchyData on PX volumes] (https://github.com/CrunchyData/crunchy-containers/tree/master/examples/portworx)
 * Use our [pxctl CLI ](https://github.com/portworx/px-dev/blob/master/cli_reference.md) to directly: 
  * View the cluster global capacity and health
  * Create, inspect, and delete storage volumes
  * Attach policies for IOPs prioritization, maximum volume size, and enable storage replication
 * Refer to the [Technical FAQ and Troubleshooting guide](https://github.com/portworx/px-dev/blob/master/faq.md) if you run into an issue. Please also feel free to [Contact us](https://github.com/portworx/px-dev#contact-us) as well. 
  

## Architecture and Storage
Portworx storage is deployed as a container and runs on a cluster of servers. Application containers provision storage directly through the Docker [volume plugins](https://docs.docker.com/engine/extend/plugins_volume/#command-line-changes:be52bcf493d28afffae069f235814e9f) API or the Docker [command-line](https://docs.docker.com/engine/extend/plugins_volume/#command-line-changes:be52bcf493d28afffae069f235814e9f). Administrators and DevOps can alternatively pre-provision storage through the Portworx command-line tool (pxctl) and then set storage policies using the Portworx administrative interface.

Portworx storage runs in a cluster of server nodes. 
 * Each server has the PX-Dev container and the Docker daemon.
 * Servers join a cluster and share configuration through the key/value store, such as etcd.
 * The PX-Dev container pools the capacity of the storage media residing on the server. You easily select storage media through the [config.json](https://raw.githubusercontent.com/portworx/px-dev/master/conf/config.json) file.

See [Deployment Requirements](https://github.com/portworx/px-dev#requirements-and-limitations) for compatibility requirements.

![fig1: storage devices](https://raw.githubusercontent.com/portworx/px-dev/master/images/cluster.png)

Storage volumes are thinly provisioned, using capacity only as an application consumes it. Volumes are replicated across the nodes within the cluster, per a volume’s configuration, to ensure high availability. 

Using MySQL as an example, a PX-Dev storage cluster has the following characteristics:
 * MySQL is unchanged and continues to write its data to /var/lib/mysql.
 * This data gets stored in the container’s volume, managed by PX-Dev. 
 * PX-Dev synchronously and automatically replicates writes to the volume across the cluster.

![fig2: MySQL volumes](https://raw.githubusercontent.com/portworx/px-dev/master/images/mysql.png)

Each volume specifies its request of resources (such as its max capacity and IOPS) and its individual requirements (such as ext4 as the file system and block size). 

Using IOPS as an example, a team can chose to set the MySQL container to have a higher IOPS than an offline batch processing container. Thus, a container scheduler can move containers, without losing storage and while protecting the user experience.

# Contact Us
As you use PX-Dev, please share your feedback and ask questions. Find the team on [Google Groups](https://groups.google.com/forum/#!forum/portworx).

If your requirements extend beyond the scope of PX-Dev, then please contact [Portworx](http://portworx.com/contact-us) for information on PX-Enterprise.


# Reference
## Description of Config.json 

| Field     | Description                                                                                                    | Example                              | Required |
|-----------|----------------------------------------------------------------------------------------------------------------|--------------------------------------|----------|
| ClusterID | A unique identifier for your cluster. Be sure to use the same identifier on all the nodes you want to cluster. | 5ac2ed6f-7e4e-4e1d-8e8c-3a6df1fb61a5 | required |
| mgtiface  | The network interface for management data.                                                                     | eth0                                 | optional |
| dataiface | The network interface for data transfers.                                                                      | eth1                                 | optional |
| kvdb      | The list of URIs to your etcd server cluster.                                                                                   | https://myetcd.example.com:4001      | required |
| devices   | The list of devices that PX-Dev will use. Any disks listed will be reformatted for PX use.                    | /dev/xvda                            | required |


## Requirements and Limitations
It is highly recommended that you run PX-Dev on a system with at least 4GB RAM.

|Requirement | Notes |
|---------------|---------|
|Kernel Version|3.10 or higher|
|Docker Version|1.10 or higher|
|KV Database|Etcd 2.0 or higher.  See https://github.com/coreos/etcd or try a hosted version at https://compose.io/etcd/|
|CPU|4 cores recommended|
|Memory|4GB Minimum|
|Cloud|If running in the cloud, AWS Ubuntu 14.04 LTS (HVM) CentOS7 with Updates HVM|
|systemd|If using systemd, Docker should NOT be set to MountFlags=slave.  PX-Dev exports mount points and requires shared mount flags.  Tracking [Docker issue 19625](https://github.com/docker/docker/issues/19625).|

Other limitations:

| Resource | Limit |
|------------|-------|
| Cluster Size | 3 |
| Per Volume Limit | 1TB |
| Max Volumes | 256 |
| Max local devices | 3 |

