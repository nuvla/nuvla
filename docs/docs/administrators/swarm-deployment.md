---
layout: default
title: Infrastructure Deployment
parent: Administrators
nav_order: 1
---

Infrastructrue Deployment
=========================

Firewall Rules
--------------

SixSq manages and maintains its public Nuvla service: 

 - URL: https://nuv.la
 - IPv4: 89.145.167.118
 
All the features developed for the GNSS Big Data CCN are included in
the production release of SlipStream and have been deployed on Nuvla.

ESA firewall rules **must** be setup to allow inbound access to the
Docker Swarm API and to the Minio S3 interface (if deployed).

End users **must** register for accounts on Nuvla to use the embedded
Nuvla interface.  Accounts are free. 


Swarm (ESA)
-----------

ESA must deploy and maintain a Docker Swarm cluster on-site as a
computational backend for applications deployed through Nuvla.

Follow any of the standard guides for installing Docker Swarm. SixSq
deploys its Swarm infrastructure on Ubuntu 16.04, but any functioning
Swarm infrastructure is acceptable.

The size and number of worker nodes to deploy depends entirely on the
foreseen workload. The test Docker Swarm cluster that has been used
for demonstrations consists on 1 master and 2 workers, all of which
have 4 vCPUs, 8 GB RAM, and 20 GB disk.

The Swarm deployment must be configured to use TLS. You must generate
at least one client certificate (and more likely one per user) to
allow authenticated access to the API.  The process we follow to do
this is:

```sh
DOCKER_TLS=/etc/docker/tls

mkdir -p $DOCKER_TLS
cd $DOCKER_TLS

openssl genrsa -aes256 -out ca-key.pem -passout pass:$PASSPHRASE 4096
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem -passin pass:$PASSPHRASE -subj "/C=CH/L=Geneva/O=SixSq/CN=$HOSTNAME"
openssl genrsa -out server-key.pem 4096
openssl req -subj "/CN=$HOSTNAME" -sha256 -new -key server-key.pem -out server.csr

echo subjectAltName = DNS:$HOSTNAME,IP:10.10.10.20,IP:127.0.0.1 > extfile.cnf
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem   -CAcreateserial -out server-cert.pem -extfile extfile.cnf -passin pass:$PASSPHRASE

# Generate client credentials
openssl genrsa -out key.pem 4096
openssl req -subj '/CN=client' -new -key key.pem -out client.csr

echo extendedKeyUsage = clientAuth > extfile.cnf
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem   -CAcreateserial -out cert.pem -extfile extfile.cnf -passin pass:$PASSPHRASE

# cleanup
rm -v client.csr server.csr
chmod -v 0400 ca-key.pem key.pem server-key.pem
chmod -v 0444 ca.pem server-cert.pem cert.pem

# configure docker drop-in
mkdir -p /etc/systemd/system/docker.service.d/
cat >>/etc/systemd/system/docker.service.d/docker-tcp.conf <<EOF
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --tlsverify --tlscacert=$DOCKER_TLS/ca.pem --tlscert=$DOCKER_TLS/server-cert.pem --tlskey=$DOCKER_TLS/server-key.pem -H=0.0.0.0:2376 -H unix:///var/run/docker.sock
EOF

systemctl daemon-reload
```

Each user must create a "docker cloud credential" resource within
Nuvla to access the Docker Swarm infrastructure through Nuvla.  If
desired, these credentials can be shared with multiple users.

The firewall must be opened to allow Nuvla inbound access to the
Docker Swarm API endpoint.  Typically this is running on the port
2376, although it can be configured on any port.

Once the Docker Swarm cluster is operational, contact SixSq Support
(support@sixsq.com) to arrange for the creation of a "cloud connector"
within Nuvla.  This will allow applications to be deployed to the
cluster through Nuvla. You will also need to provide a client
credential that allows access to the Swarm API endpoint for testing
and monitoring purposes.


NFS (ESA)
---------

The GNSS data "objects" must be stored within an NFS infrastructure to
allow those objects to be mounted on containers as volumes.  Although
optional, we recommend that these object also be made available via S3
(see Minio section below).

Deploy a standard NFSv4 server.  The size of the server (or cluster)
depends on the volume of data that you expect to host.

Configure the NFS server to allow all the nodes in the Swarm cluster
to mount directories from the server.  Example entries in the
`/etc/exports` file are:

    /data/gnss-gnss-swarm-20181201t230000z *(ro,sync,no_subtree_check)
    /data/minio *(rw,sync,no_subtree_check)

Here the access is controlled via a firewall, allowing the use of
wildcard rules.  Alternatively, the hosts can be listed explicitly in
the `/etc/exports` file.

If using Minio for an S3 interface, you must allow that service to
access the NFS server as well.


Minio (ESA)
-----------

Minio (https://www.minio.io/) is a container-based service that
provides a compliant S3 interface for a number of storage backends.
Here, we recommend that Minio be deployed with the NFS server as the
storage backend.

Deploying this service is optional. However, by deploying this, it
allows users to take advantage of the SlipStream's "external object"
resources, which among other features, allows convenient upload and
download of data objects.

Follow the standard installation instructions for Minio using the NAS
backend.  You will need to allow Minio client access to your NFS
server.
