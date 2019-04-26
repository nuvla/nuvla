---
layout: default
title: Docker Swarm Infrastructures
parent: Administrators
nav_order: 2
---

Docker Swarm Infrastructures
============================

There are two types of Docker Swarm infrastructures that can be
deployed for Nuvla:

 - **target**: A Docker Swarm infrastructure on which Nuvla will
   **deploy** containers.  This is a basic deployment of Swarm with
   Minio/NFS for data access and Prometheus for monitoring. Nuvla can
   be configured to deploy to any number of target Docker Swarm
   infrastructures.

 - **host**: A Docker Swarm infrastructure that will **host** a Nuvla
   deployment. This is a basic deployment of Swarm with Prometheus for
   monitoring.

In both cases, volumes must be used for data persistency in production
environments.

Before starting, review the entire installation procedure.  The
[nuvla/deployment](https://github.com/nuvla/deployment) GitHub
repository contains files that will help you deploy and configure your
Docker Swarm cluster.  You may need to customize the provided scripts,
configurations, and Docker Compose files for your deployment.

## Docker Swarm Cluster

Docker Swarm clusters provide the computational platforms that Nuvla
uses to deploy container-based applications, including Nuvla itself.

Any method can be used to deploy a Docker Swarm cluster.  See the
[Docker Swarm documentation](https://docs.docker.com/engine/swarm/)
for an overview of Docker Swarm and how to deploy it.

You may want to consider [Docker
Machine](https://docs.docker.com/machine/) for installation; it
automates the deployment of a Docker Swarm cluster on a cloud
infrastructure.

The nuvla/deployment repository contains a script
(`deploy-swarm-exoscale.sh`) to deploy a Swarm cluster with Docker
Machine. Clone this repository:

    git clone https://github.com/nuvla/deployment.git

to a machine that will be convenient for managing your cluster.

This script uses Docker Machine to deploy a Swarm cluster on the
[Exoscale](https://exoscale.ch) cloud. Customize this script to change
the cloud driver or the sizes of machines deployed.

In your cloned repository, descend into the `swarm` subdirectory and
copy `env-example.sh` to `env.sh`. Edit this file, changing the values
of the variables to customize your installation. The `SSH_KEY` and
`EXOSCALE_*` variables are the most important for the Swarm
deployment.

Docker Machine uses SSH to communicate with the virtual machines of
the cluster. By default the key `${HOME}/.ssh/id_rsa` will be used (or
created if it does not exist). If you want to use a different key,
then set the environmental variable `SSH_KEY` in the `env.sh` file.

> **WARNING**: Use an SSH key **without** a password. If you use one
> with a password, you will be prompted for it, repeatedly. To
> generate a new SSH key without a password just set SSH_KEY to a file
> that does not exist.

After your changes, run:

    source env.sh

to set all of the environmental variables for the Swarm management
script. 

The command to use to create the cluster is:

    ./swarm-deploy-exoscale.sh deploy 3

This creates a cluster with one master and two workers (three nodes in
total). If you do not provide the second argument, it defaults to one.

The size and number of worker nodes to deploy depends entirely on the
foreseen workload. The script uses "Large" instances that have 4
vCPUs, 8 GB RAM, and 50 GB disk.

You will want to note the IP addresses of the Docker Swarm master and
workers (if any). You can recover these IP addresses by running the
command `docker-machine ls` if necessary.

You can access your machines with:

    docker-machine ssh dockermachine-1556097484
    
using the name of the machine provided in the machine listing. This
may be a non-root account; to become root use the command `sudo su -`.

To tear down the Swarm cluster use the command:

    ./swarm-deploy-exoscale.sh terminate

will terminate all the machines in your Swarm cluster. Use with care.

## Cluster Firewall Rules

For **target** infrastructures, the Nuvla service **must** be able to
access the services on the cluster. The site's firewall rules must be
setup to allow **inbound access** to the Docker Swarm API and to the
HTTP(S) ports for other services, e.g. Minio S3.  The nodes in the
cluster must also be able to communicate between themselves.

In addition, you will want to allow users to access applications that
they've deployed on the infrastructure. To do this, make sure that
**inbound access** is possible for all ephemeral ports from anywhere
(or at least from all IPs addresses of your users). The range of
ephemeral ports can be configured via the kernel parameter
`/proc/sys/net/ipv4/ip_local_port_range`.

The following table shows the ports to open for **target**
infrastructures:

| port | service | inbound addresses |
| --- | --- | --- |
| 22 (TCP) | ssh |  management nodes |
| 8080 (TCP) | traefik |  management nodes | 
| 4789 (UDP), 7946 (UDP, TCP) | docker |  cluster nodes |
| 111 (UDP, TCP), 2049 (UDP, TCP), 33333 (UDP, TCP) | nfs |  cluster nodes |
| 2376-2377 (TCP) | docker |  Nuvla |
| 80 (TCP), 443 (TCP) | http(s) |  Nuvla, users |
| 32768-60999 (TCP) | ephemeral ports |  users |

A **host** infrastructure has fewer ports that need to be opened, as
Nuvla is generally the only publicly accessible service on the system.

The following table shows the ports to open for **host**
infrastructures:

| port | service | inbound addresses |
| --- | --- | --- |
| 22 (TCP) | ssh |  management nodes |
| 8080 (TCP) | traefik |  management nodes | 
| 4789 (UDP), 7946 (UDP, TCP) | docker |  cluster nodes |
| 2376-2377 (TCP) | docker |  management nodes |
| 80 (TCP), 443 (TCP) | http(s) |  management nodes, users |

> **NOTE**: If you used Docker Machine to deploy your cluster on a
> cloud, the driver will likely have created a security group.  You
> can modify this security group, if necessary to expose non-Docker
> ports, e.g. the ephemeral ports for applications.

## Create Public Network

The various components (both Nuvla components and other required
components) will use an "external" network when making services
available outside of the Docker Swarm cluster. 

Create a public overlay network by running the following command on
the master node of the Swarm cluster:

    docker network create --driver=overlay traefik-public

The name "traefik-public" is hardcoded in many of the docker-compose
files. If you want to use a different name, you'll need to search for
the "traefik-public" references and update those files.

> **NOTE**: For debugging purposes, you may want to add the
> `--attachable` option when creating the network.  This allows you to
> start containers on the network and directly access the services
> there. Generally, you should not use this option in production.

## Deploy Traefik

Traefik is a general router and load balancer.  Clone the
nuvla/deployment repository to the master node of your Docker Swarm
cluster.

There are two versions of the traefik deployment: traefik and
traefik-prod. The first, suitable for test or demonstration
deployments, uses a self-signed certificate for connections to the
HTTPS (443) port; the second, suitable for production, uses Let's
Encrypt certificates.

### Self-Signed Certificates

To deploy an infrastructure for test or demonstration purposes, use
the traefik configuration that uses self-signed certificates. You can
find this in the `swarm/traefik` subdirectory of the repository. 

To deploy traefik do the following on the master node of the cluster:

    cd traefik
    ./generate-certificates.sh
    docker stack deploy -c docker-compose.yml traefik

This will generate temporary, self-signed certificates and bring up
traefik.  You can view the current configuration of traefik at the URL
http://master-ip:8080; this is updated dynamically as services are
added and removed.

### Let's Encrypt

To deploy a production infrastructure with a certificate obtained from
[Let's Encrypt](https://letsencrypt.org/), use the configuration that
can be found in the `swarm/traefik-prod` subdirectory.

You will need to have access to an internet domain to use Let's
Encrypt certificates. See the [Let's Encrypt
documentation](https://letsencrypt.org/docs/) for instructions on how
to configure your (sub)domain.

Modify the configuration file
`swarm/traefik-prod/traefik/traefik.toml`, to add the name of your
domain and your email address. Once that's done, you can deploy
traefik with:

    cd traefik-prod
    chmod 0600 traefik/acme.json 
    docker stack deploy -c docker-compose.yml traefik

After deploying traefik, verify that the correct certificate is being
used. Traefik store the generated certificate and private key in
following file `traefik-prod/traefik/acme.json`. Take a look. If
something goes wrong, take a look at the traefik logs.

## Monitoring

Having an overview of the activity on the Docker Swarm cluster is
extremely helpful in understanding the overall load and for diagnosing
any problems that arise. We recommend using Prometheus to monitor the
cluster.

To deploy Prometheus with the standard configuration (from a cloned
version of the nuvla/deployment repository), run the command on the
Swarm master:

    cd monitoring
    docker stack deploy -c docker-compose.yml prometheus

The service will be available at the URL `https://master-ip/prometheus`.
The following services will appear:

| service | URL |
| --- | --- |
| grafana | https://master-ip/grafana | monitoring dashboard |
| prometheus | https://master-ip/prometheus/ | Prometheus administration |

Normally, you will only be interested in the Grafana dashboard, which
provides a visual overview of the Swarm cluster operation.

## NFS

To allow for user data "objects" to be accessed via POSIX from within
containers, the nodes hosting the Docker Swarm cluster must have NFS
installed.  This is optional, but strongly recommended.

If you are deploying a **target** infrastructure, then be sure that
**all** nodes have the NFS client software installed.

This can be done, for example on Ubuntu, by accessing the nodes as
`root` via SSH and running the command:

    apt-get update
    apt-get install nfs-kernel-server

(Note: this actually installs the server as well; easier than just
installing the client alone.)

If you use one of the Swarm nodes (e.g. the master) as the NFS server,
be sure that the NFS daemon is installed there.

On the NFS server, create the directory that will be shared with all
nodes of the Swarm cluster.  The commands to do this on Ubuntu are:

    NFS_SHARE='/nfs-root'
    mkdir ${NFS_SHARE}
    chown nobody:nogroup ${NFS_SHARE}
    chmod 777 ${NFS_SHARE}
    echo -e "${NFS_SHARE} *(ro,sync,no_subtree_check)" >> /etc/exports
    exportfs -a
    systemctl enable nfs-kernel-server
    systemctl restart nfs-kernel-server

Note that this configuration allows any node within the cluster to
mount the volumes.  If the network is open to nodes outside the
cluster, you may want to provide an explicit list of allowed hosts.

## Minio (S3)

The data management services rely on the availability of an
S3-compatible service on the **target** Docker Swarm
infrastructures.

Minio is a container-based implementation that can expose NFS volumes
via the S3 protocol. For **target** infrastructures, you can deploy
Minio with:

    cd minio
    docker stack deploy -c docker-compose.yml minio

The service will be available at the URL
`https://master-ip/minio/`. (Be patient, minio takes a minute or so to
start and then traefik must adjust its configuration.) The default
username/password will be admin/admin, if you've not changed them in
the configuration.
