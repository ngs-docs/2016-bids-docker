Building up on Docker: Compose, Swarm and Carina
================================================

Please see `the README <README.md>`__ for background information.

Contact: ngs@luizirber.org

License: CC0

We'll put ephemera from today in this etherpad:

   https://etherpad.wikimedia.org/p/bids-docker


Docker Compose
==============

- Defining and running multi-container Docker applications
- Compose file (docker-compose.yml) describes how to configure all the application's services

Basic workflow
--------------

Create (or use an existing) Dockerfile for each service you'll run.

Name and configure how the services interact in docker-compose.yml.

Run ::

    docker-compose up

and Compose will start and run all the services.

Example: Compose and Django
---------------------------

https://docs.docker.com/compose/django/

* configurations to cover: build, ports, links, image
https://docs.docker.com/compose/compose-file/


Extending the docker infrastructure
===================================

- Machine (deploy basic infrastructure)

  * Single machine deploy (install Docker on an EC2 instance)

- Swarm (cluster management)

  * Connect a pool of Docker hosts into a single, virtual Docker host.
  * Expose the Docker API, so tools compatible with a Docker daemon also work with Swarm.

- Compose (Application deployment)

  * Application deployment
  * Service description and relationships

* talk about how this development is iterative (and how sometimes things don't work that well)
* diagram everything!

Other toolchains
----------------

  * Kubernetes
  * CoreOS (etcd, fleet, ...)

Useful for science?
===================



