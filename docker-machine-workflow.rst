==========================================================
Building and running Docker containers with docker-machine
==========================================================

:author: \C. Titus Brown, titus@idyll.org
:license: CC0
:date: Jan 8. 2016

Below, we'll build a Docker container with the MEGAHIT assembler installed,
and then execute it on some Real Data.

.. contents::

Preparation
===========

Spin up a Docker host on a reasonably sized computer (e.g. see "Using
docker-machine" instructions in `day 1 <day.rst>`__ to spin up a
docker host on EC2).

Build an image
==============

Let's start by building an image for the MEGAHIT assembler.

Building an image with a Dockerfile
-----------------------------------

To build and run MEGAHIT, we need to do the following:

* install g++, make, git, zlib1g-dev, and python
* clone megahit from https://github.com/voutcn/megahit
* build it

In Dockerfile format, this is::

   FROM ubuntu:14.04
   RUN apt-get update && apt-get install -y g++ make git zlib1g-dev python
   RUN git clone https://github.com/voutcn/megahit.git /home/megahit
   RUN cd /home/megahit && make
   CMD /mydata/do-assemble.sh

Here, the 'FROM' command tells Docker what container to load; the 'RUN'
commands tell Docker what to execute (and then save the results from);
and the `CMD` specifies the script entry point - a command that is
run if no other command is given.

Let's build a Docker image from this and see what happens! Put those commands
in a Dockerfile in its own directory, cd into that directory, and then run::

   docker build -t megahit_ctr .

(This will take a few minutes.)

You should see the resulting docker image with::

   docker images | grep megahit

Once it's built, you can now run it like so::

   docker run -it megahit_ctr /home/megahit/megahit

...and voila! You are running megahit!

Connecting a Docker container to some external data
---------------------------------------------------

The next big problem here is that we don't actually have any data
"close" to the Docker image.  If we were running this on a Docker host
on our laptop with virtualbox, we could mount a host directory with '-v' --
but here, we don't have that option.

So instead let's create a data volume! ::

  docker create -v /mydata --name my_data_vol ubuntu:14.04 /bin/true

and then populate it with data::

  docker run --rm --volumes-from my_data_vol -it megahit_ctr /bin/bash

Inside the docker container, run

  apt-get install wget
  cd /mydata
  wget http://public.ged.msu.edu.s3.amazonaws.com/ecoli_ref-5m-trim.pe.fq.gz
  wget http://public.ged.msu.edu.s3.amazonaws.com/ecoli_ref-5m-trim.se.fq.gz

This downloads those two data files onto your /mydata directory, which
is (equivalently) 'my_data_vol'.

Also in the docker container, let's run::

  cd /mydata
  cat > do-assemble.sh <<EOF
  /home/megahit/megahit --12 /mydata/*.pe.fq.gz \
                         -r /mydata/*.se.fq.gz \
                         -o /mydata/ecoli -t 4
  EOF
  chmod +x do-assemble.sh

This creates the script that we're using as a CMD in the Dockerfile.

Now, exit the docker container and run::

  docker run --rm --volumes-from my_data_vol -it megahit_ctr

...and you should see the assembler hard at work.  You can
copy files directly off of my_data_vol with 'docker cp'.

One thing to note here is that we've placed the ``do-assemble.sh`` script on
the data volume, rather than in the megahit_ctr container.

The con of this is that the megahit container isn't as self-sufficient as
it would be if you just had a generic processing script that did stuff.

The pro of this is that you can write your own processing script and
keep it with your data.  And you didn't have to write a generic
processing script :).

An alternative using local volumes
----------------------------------

You can also do something similar with local volumes --
'docker-machine ssh' into your host, create a directory somewhere,
download the data into that directory and create the do-assemble script,
and then add::

    -v /docker/host/directory:/mydata

to the 'docker run' command above.
