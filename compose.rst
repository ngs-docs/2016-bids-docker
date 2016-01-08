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

Compose installation
---------------------

If you used Docker toolbox, you already have it!

Other installation options, check:
https://docs.docker.com/compose/install/

Example: Compose and Django
---------------------------

https://docs.docker.com/compose/django/

::
  mkdir -p example
  cd example

Create a Dockerfile ::

    FROM python:2.7
    ENV PYTHONUNBUFFERED 1
    RUN mkdir /code
    WORKDIR /code
    ADD requirements.txt /code/
    RUN pip install -r requirements.txt
    ADD . /code/


Create requirements.txt ::

    Django
    psycopg2

Create docker-compose.yml ::

	db:
	  image: postgres
	web:
	  build: .
	  command: python manage.py runserver 0.0.0.0:8000
	  volumes:
		- .:/code
	  ports:
		- "8000:8000"
	  links:
		- db

Create a new Django project inside the web container ::

    $ docker-compose run web django-admin.py startproject composeexample .

Fix permissions ::

	sudo chown -R $USER:$USER .

Edit composeexample/settings.py ::

	DATABASES = {
		'default': {
			'ENGINE': 'django.db.backends.postgresql_psycopg2',
			'NAME': 'postgres',
			'USER': 'postgres',
			'HOST': 'db',
			'PORT': 5432,
		}
	}

Bring up all the services ::

    $ docker-compose up

* configurations to cover: build, ports, links, image: https://docs.docker.com/compose/compose-file/
* upcoming changes with compose 1.6


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

* diagram everything!

Other toolchains
----------------

  * Kubernetes
  * CoreOS (etcd, fleet, ...)

Carina
======

- Rackspace service based on Docker Swarm
- Currently in Beta (no charges!)
  * But limited to 3 instances
- Simple interface for cluster creation

https://getcarina.com/docs/getting-started/getting-started-carina-cli/

* talk about how Docker development is iterative (and how sometimes things don't work that well...)

Useful for science?
===================
