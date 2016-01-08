Docker @ BIDS - Day 2 notes
===========================

Please see `the README <README.md>`__ for background information.

Contact: titus@idyll.org

License: CC0

We'll put ephemera from today in this etherpad:

   https://etherpad.wikimedia.org/p/bids-docker

mybinder.org
------------

mybinder provides a (demo) service that will spin up a Jupyter
notebook in a Docker container, with the option of specifying your own
configuration (via a Python requirements.txt, an Anaconda
environment.yml, or a Dockerfile (!).

Importantly, other files in the github repo come with it so you
can include data!

1. To try it out, go to mybinder.org and enter

     https://github.com/ctb/2016-mybinder-inflammation

   with no dependencies, then click 'make my binder'.

   Once building is completed, click on 'launch binder'.
   You'll be sent to an IPython Notebook that you can just run.
   Here, the data set to be loaded is included, because it's
   in the github repo!

   If you want to try customizing it, edit the IPython Notebook,
   download it, fork the 2016-mybinder-inflammation repo, add
   the updated notebook into that repo, and rerun mybinder.org on
   your fork -- you will see the new notebook in there!

2) For a simple demo of just how powerful/flexible Docker can make things,
   checkout this repo:

      https://github.com/ctb/2016-mybinder-irkernel

   Here, I'm *installing R* and the R-Jupyter bridge inside the Dockerfile,
   and then using that to run an R notebook!

   You can give it a try by entering the above URL into the mybinder.org site,
   and specifying 'Dockerfile' under 'Configure dependencies'.

----

Also see 'everware', which I haven't had time to try:

   https://github.com/everware
  
-----

Hands-on data intensive workflow stuff
--------------------------------------

`Setting up a data-intensive workflow on docker-machine <docker-machine-workflow.rst>`__
