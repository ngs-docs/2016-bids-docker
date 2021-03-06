Docker @ BIDS - Day 1 notes
===========================

Please see `the README <README.md>`__ for background information.

Contact: titus@idyll.org

License: CC0

We'll put ephemera from today in this etherpad:

   https://etherpad.wikimedia.org/p/bids-docker

Introductory stuff
------------------

Who am I? Currently, background, interests...

My meta-goals for this workshop: hands-on exposure, with lots of rooms
for questions. Please ask questions.

Note: I'm not a Docker expert by any means.

Caveat: this is the first time I'm teaching a lot of this; please be
gentle!

Starting level.

Other instructors: I will be joined by Carl Boettiger (today) and
Luiz Irber (tomorrow).

We have a `code of conduct! <http://software-carpentry.org/conduct/>`__
Basically, please be respectful of each other.

All of this material will be open (CC0). Use it as you wish!

Pretty please, ask questions!

Notes on sticky notes.

Bathrooms!

A few likely challenges:

* Linux, Mac, Windows... Amazon?
  
* Network bandwidth

**What are your goals and interests for this workshop?**

Before Getting Started
----------------------

Did everyone get an e-mail from me this morning?
 
Can you run this? ::

   docker run hello-world

and see something like this? ::

   Hello from Docker.
   ...

If that works, you should do::

   docker pull ubuntu:14.04

If that doesn't work, you `might need
<https://stackoverflow.com/questions/28301151/docker-pull-error>`__ to
run this to reset your local docker install::

   docker-machine restart default && eval "$(docker-machine env default)"

You can also download these bigger images, that we will be using later::

   docker pull jupyter/notebook
   docker pull rocker/hadleyverse

Getting started
---------------

Try::

   docker run ubuntu:14.04 /bin/echo 'Hello world'

Now try::

   docker run -it ubuntu:14.04 /bin/bash

Q: What does the '-it' do?

Points to cover:

* you can use CTRL-D or type 'exit' to leave the container
* "images" are the disk, "containers" are the running bit

----

Try creating a file in your container; first run::

   docker run -it ubuntu:14.04 /bin/bash

and then *inside the docker container* run::

   echo hello, there > /home/foo.txt

Now, exit the container.  Make note of the container ID if you can -
it's the string right after 'root@' and before the ':', and should be
something like '003eafc0422c'.

First, if you run 'docker run ubuntu:14.04 -it /bin/bash' you will see
that the file is not there::

   cat /home/foo.txt

This is because every container starts fresh from the base image (here,
ubuntu:14.04).

However, you can still access files from old containers (unless you specify
'--rm' when you run things). Here, ::

   docker cp 003eafc0422c:/home/foo.txt .

will copy the file into your current directory.
  
So: data is transient, unless you make other provisions (we'll talk
about this later!)

----

You can get a list of running containers by doing::

  docker ps

You can get a list of *all* containers (running and stopped) by doing::

  docker ps -a

You can get a list of all *images* by doing::

  docker images

You can remove a container with 'docker rm', and remove an image with
'docker rmi'.  The 

Two handy commands to clean up (a) stopped containers and (b)  ::

  docker rm $(docker ps -a -q)
  docker rmi $(docker images | grep "^<none>" | awk "{print $3}")

Other things to be sure to mention:
  
* talk a bit about `layerfs / Union file systems <https://docs.docker.com/engine/introduction/understanding-docker/#union-file-systems>`__
* you can run most any Linux install, although I mostly use Ubuntu.

Doing something useful -- running some complicated software
-----------------------------------------------------------

Let's try running a Jupyter notebook server::

  docker run -it -p 9000:8888 -v ${PWD}/nb:/notebooks jupyter/notebook

Then connect to ``http://<ip address>:9000/``.

This may be tricky --

* if you're running on Mac or Windows, you'll find the ``<ip address>``
  with ``docker-machine ip default``.

* if you're running locally on a Linux machine, you can use 127.0.0.1.

* if you're running remotely on an Amazon machine, you need to use the
  IP address of the Amazon machine, and you *also* need to open up port
  9000 on the Amazon machine.

Things to touch on:

* mirroring local volumes with '-v from:container'
* connecting ports with '-p from:container'
* ``docker port <container_id>``
* naming containers with ``--name``

----

You can also do this with RStudio::

  docker run -it -p 9001:8787 --name rstudio rocker/hadleyverse

and then go to ``http://$(docker-machine ip default):9001``.

* note that you can use '-d' here instead of '-it', but then you have
  to kill the container with 'docker rm'.

Some challenge exercises:
  
* If you run `getwd()` in R you'll see that here you're in
  '/home/rstudio'; how would you map your local directory into the
  container?

* If you don't map your local directory, how would you copy R scripts off?

----

Bigger questions:

* how do I figure out what all the arguments do?
* why is this useful, anyway?

Docker data volumes
-------------------

If you run::

   docker create -v /mydata --name my_data_vol ubuntu:14.04 /bin/true

you now have a volume that persists across docker containers.  You can
mount it like so::

   docker run --volumes-from my_data_vol -it ubuntu:14.04 /bin/bash

This is convenient for many reasons: it persists across reboots,
and you can copy to and from it by name::

   docker cp my_data_vol:/mydata/filename.txt .

You can remove it with::

   docker rm my_data_vol

Things to discuss:

* discuss where data volumes (and images, etc.) are stored
   
Building a new Docker image
---------------------------

Create a subdirectory 'firsttry' and put a file named Dockerfile in there,
containing::

  FROM ubuntu:14.04
  RUN echo 'echo hello, world' > /home/hello.sh && chmod +x /home/hello.sh
  CMD /home/hello.sh

Then, cwd in that directory, do::

  docker build -t myhello .

You can now run::

  docker run myhello

More::

* the "local context" (files in the cwd) are copied over to the host machine
* commands to cover: FROM, COPY, RUN, ENV, WORKDIR, CMD.
* A real Dockerfile is at: https://github.com/ctb/2015-docker-building/tree/master/khmer
* note, each RUN command creates a new layer...
* are people interested in docker hub?

Using docker-machine
--------------------

Documentation: https://docs.docker.com/machine/; also see `Amazon Web
Services driver docs <https://docs.docker.com/machine/aws/>`__

Here, we're going to use Amazon to host and run our Docker images, while
controlling it from our local machine.

Start by logging into the `AWS EC2 console <https://console.aws.amazon.com/ec2/v2/home>`__.

Find your AWS credentials and your VPC ID.

* your AWS credentials are `here <https://console.aws.amazon.com/iam/home?region=us-east-1#security_credential>`__, and if you haven't used them before you
  may need to "Create a New Access Key".  (Be sure not to store these in a place
  that other people can view them.)

  Your AWS

* to get your VPC ID, go into https://console.aws.amazon.com/vpc/home and
  select "Your VPCs".  Your VPC ID should look something like vpc-9efe1afa
  (that's mine and won't work for you ;)

Then, set your AWS_KEY and AWS_SECRET and VPC_ID; on Linux/Mac, fill in
the values and execute::

  export AWS_KEY=
  export AWS_SECRET=
  export VPC_ID=

...not sure what to do on Windows, maybe build the command below in a text
editor?

Then, run::
  
  docker-machine create --driver amazonec2 --amazonec2-access-key ${AWS_KEY} \
        --amazonec2-secret-key ${AWS_SECRET} --amazonec2-vpc-id ${VPC_ID} \
        --amazonec2-zone b --amazonec2-instance-type m3.xlarge \
        aws

and to connect to it, do::

  eval $(docker-machine env aws)

and now you can run all the 'docker' commands as you would expect, EXCEPT
that your docker host is now running Somewhere Else.

If you have trouble getting a subnet, make sure that your VPC has subnets
in the zone/region you're trying to use. You can set these with::

    --amazonec2-region "us-east-1" --amazonec2-zone b

Take a look at the help for the EC2 driver here::

  docker-machine create --driver amazonec2 -h | less

Things to discuss:

* diagram out what we're doing!
* docker-machine manages your docker host; docker manages your
  containers/images ON that host.
* talk about AWS host sizes/instance types: https://aws.amazon.com/ec2/instance-types/
* explain docker client, docker host, docker container relationship
* also include -p, -v discussion.

---

You can use 'docker-machine stop aws' and 'docker-machine start aws' to
stop and start this machine; with AWS, you will need to do a
'docker-machine regenerate-certs aws' after starting it in order to
connect to it with docker-machine env.

To kill the machine, do 'docker-machine kill aws'.  This will also, I believe,
trash the configuration settings so you would need to reconfigure it
with a 'create'.

Note that while the machine is running or stopped, you should be able
to see it at the `AWS EC2 console
<https://console.aws.amazon.com/ec2/v2/home>`__.

----

Let's talk more about why you would want to do *this* :).

Also, diagrams!

Also, note Dockerfiles build on the *remote* machine!

-----

You might also be interested in running through the full
`docker-intro on Amazon <docker-intro.rst>`__; we can do it in class
if we have time.  This is a more complete workflow for how I intend to
user docker myself.

Some challenge exercises
------------------------

Rewrite the 'myhello' Dockerfile above to copy a pre-created hello.sh into
the container, rather than creating it with 'echo'.

----

Write a Dockerfile that starts from jupyter/notebook but configures
the /notebooks directory to start with some files.

Equivalently, write a Dockerfile that starts from rocker/hadleyverse but
configures the /home/rstudio directory to start with some files.

----

Create a data volume that persists /notebooks across jupyter/notebook
runs (OR equiv, that persists /home/rstudio across rocker/hadleyverse
runs).

Thoughts on containerization, scientific workflows, etc
-------------------------------------------------------

- data volumes vs local disk; see my reasoning `here <http://ivory.idyll.org/blog/2015-transcriptomes-with-docker.html>`__
- what about putting data, scripts on persistent volume and using Docker
  containers for the base software?
- standardization and packaging... bioboxes!
- the long-term idea of binding data resources to Docker containers and/or
  specify workflows with a directed acyclic graph...

.. bioboxes presentation?
