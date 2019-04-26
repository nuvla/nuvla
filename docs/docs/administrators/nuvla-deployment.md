---
layout: default
title: Nuvla Deployment
parent: Administrators
nav_order: 1
---

Nuvla Deployment
================

The Nuvla services and their dependent services are all containerized
and can be deployed easily into a Docker Swarm cluster. The deployment
has been split into two groups to facilitate the management of the
services:

 - **db**: Deploys the database elements of the Nuvla software stack,
   which consists of Elasticsearch and Zookeeper. 

 - **core**: Deploys the "business logic" of Nuvla, which consists of
   a number of micro-services.

A graphical overview of the deployment can be seen below.

![Nuvla Deployment]({{ site.url }}{{ site.baseurl }}/docs/administrators/assets/overview.png)

Before starting, review the entire deployment procedure.  The
[nuvla/deployment](https://github.com/nuvla/deployment) GitHub
repository contains files that will help you deploy Nuvla.

## Prerequisites

Review the requirements for the **host** Docker Swarm cluster that you
will use to deploy Nuvla. These can be found in the [Docker Swarm
Infrastructures](swarm-deployment.html) section. In particular, **be
sure that you have created the "traefik-public" network and that
Traefik has been deployed**.

## Backend Network

The Nuvla services and their dependencies communicate over a network
that is not accessible publicly. Because this is shared between
separate deployments, it must be created manually. 

Create the private, backend overlay network by running the following
command on the master node of the Swarm cluster:

    docker network create --driver=overlay nuvla-backend

Verify that the network has been properly created by using `docker
network ls`.

## Database

Nuvla uses Elasticsearch to store the vast majority of its state,
including all the stored documents that represent resources in the
system. Zookeeper stores the remaining state, which consists of the
states of asynchronous jobs in the system.

**Before** trying to deploy Nuvla's core services, first start the
database elements.  To do so, clone the nuvla/deployment repository on
the Docker Swarm master.

Navigate to the "prod/db" directory in your checked out
repository. There you execute the command:

    docker stack deploy -c docker-compose.yml db

Verify that the Elasticsearch and Zookeeper services come up
correctly, by listing them with Docker (`docker service ls`) or by
inspecting the logs.

## Core Services

Once the database services have been deployed, you can then deploy the
core services for Nuvla.  In your clone of the nuvla/deployment
repository, navigate to the "prod/core" directory. To do so, execute
the commands:

    ./generate-certificates.sh
    docker stack deploy -c docker-compose.yml nuvla

Verify that all of the services start by listing them with Docker.

You should also verify that the API server has fully started by
inspecting the logs:

    docker service logs -f nuvla_api

After the configuration messages, the log should indicate that a
server has been started on port 8200. Once this is the case, you can
visit https://master-ip/api/cloud-entry-point, which should return a
directory of the available resources in JSON format.

![Cloud Entry Point]({{ site.url }}{{ site.baseurl }}/docs/administrators/assets/cloud-entry-point-json.png)

Similarly, you can verify that the web browser interface is available
by visiting https://master-ip/. You should see the Nuvla UI with a
login button in the upper-right corner.

![Nuvla Welcome Page]({{ site.url }}{{ site.baseurl }}/docs/administrators/assets/nuvla-welcome.png)

