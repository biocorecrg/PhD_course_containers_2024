.. _docker_2-page:

*******************
Docker 2
*******************

Docker recipes: build your own images
=====================================

OS commands in image building
-----------------------------

Depending on the underlying OS, there are different ways to build images.

Know your base system and their packages. Popular ones:

* `Debian <https://packages.debian.org>`__

* `CentOS <https://centos.pkgs.org/>`__

* `Alpine <https://pkgs.alpinelinux.org/packages>`__

* Conda. `Anaconda <https://anaconda.org/anaconda/repo>`__, `Conda-forge <https://conda-forge.org/feedstocks/>`__, `Bioconda <https://anaconda.org/bioconda/repo>`__, etc.


Update and upgrade packages
***************************

* In **Ubuntu**:

.. code-block::

  apt-get update && apt-get upgrade -y


In **CentOS**:

.. code-block::

  yum check-update && yum update -y


Search and install packages
***************************

* In **Ubuntu**:

.. code-block::

  apt search libxml2
  apt install -y libxml2-dev


* In **CentOS**:

.. code-block::

  yum search libxml2
  yum install -y libxml2-devel.x86_64


Note the **-y** option that we set for updating and for installing.<br>
It is an important option in the context of Docker: it means that you *answer yes to all questions* regarding installation.


Building recipes
----------------

All commands should be saved in a text file, named by default **Dockerfile**.

Basic instructions
******************

Each row in the recipe corresponds to a **layer** of the final image.

**FROM**: parent image. Typically, an operating system. The **base layer**.

.. code-block::

  FROM ubuntu:18.04


**RUN**: the command to execute inside the image filesystem.

Think about it this way: every **RUN** line is essentially what you would run to install programs on a freshly installed Ubuntu OS.

.. code-block::
  RUN apt install wget


A basic recipe:

.. code-block::

  FROM ubuntu:18.04

  RUN apt update && apt -y upgrade
  RUN apt install -y wget


docker build
************

Implicitely looks for a **Dockerfile** file in the current directory:

.. code-block:: console

  docker build .

Same as:

.. code-block:: console

  docker build --file Dockerfile .


Syntax: **--file / -f**

**.** stands for the context (in this case, current directory) of the build process. This makes sense if copying files from filesystem, for instance. **IMPORTANT**: Avoid contexts (directories) overpopulated with files (even if not actually used in the recipe).

You can define a specific name for the image during the build process.

Syntax: **-t** *imagename:tag*. If not defined ```:tag``` default is latest.

.. code-block:: console

  docker build -t mytestimage .
  # Same as:
  docker build -t mytestimage:latest .


* IMPORTANT: Avoid contexts (directories) over-populated with files (even if not actually used in the recipe).
In order to avoid that some directories or files are inspected or included (e.g, with COPY command in Dockerfile), you can use .dockerignore file to specify which paths should be avoided. More information at: https://codefresh.io/docker-tutorial/not-ignore-dockerignore-2/


The last line of installation should be **Successfully built ...**: then you are good to go.

Check with ``docker images`` that you see the newly built image in the list...

Then let's check the ID of the image and run it!

.. code-block:: console

  docker images

  docker run f9f41698e2f8
  docker run mytestimage


More instructions
*****************

**WORKDIR**: all subsequent actions will be executed in that working directory

.. code-block::

  WORKDIR ~

**ADD, COPY**: add files to the image filesystem

Difference between ADD and COPY explained `here <https://stackoverflow.com/questions/24958140/what-is-the-difference-between-the-copy-and-add-commands-in-a-dockerfile>`__ and `here <https://nickjanetakis.com/blog/docker-tip-2-the-difference-between-copy-and-add-in-a-dockerile>`__

**COPY**: lets you copy a local file or directory from your host (the machine from which you are building the image)

**ADD**: same, but ADD works also for URLs, and for .tar archives that will be automatically extracted upon being copied.

If we have a file, let's say ```example.jpg```, we can copy it.

.. code-block::

  # COPY source destination
  COPY example.jpg .

A more sophisticated case:

.. code-block::

  FROM ubuntu:18.04

  RUN apt update && apt -y upgrade
  RUN apt install -y wget

  RUN mkdir -p /data

  WORKDIR /data

  COPY example.jpg .


**ENV, ARG**: run and build environment variables

Difference between ARG and ENV explained `here <https://vsupalov.com/docker-arg-vs-env/>`__.

* **ARG** values: available only while the image is built.
* **ENV** values: available during the image build process but also for the future running containers.
  * It can be checked in a resulting running container by running ``env``.

**CMD, ENTRYPOINT**: command to execute when generated container starts

The ENTRYPOINT specifies a command that will always be executed when the container starts. The CMD specifies arguments that will be fed to the ENTRYPOINT

In the example below, when the container is run without an argument, it will execute `echo "hello world"`.
If it is run with the argument **hello moon** it will execute `echo "hello moon"`

.. code-block::

  FROM ubuntu:18.04
  ENTRYPOINT ["/bin/echo"]
  CMD ["hello world"]


A more complex recipe (save it in a text file named **Dockerfile**:

.. code-block::

  FROM ubuntu:18.04

  MAINTAINER Toni Hermoso Pulido <toni.hermoso@crg.eu>

  WORKDIR ~

  RUN apt-get update && apt-get -y upgrade
  RUN apt-get install -y wget

  ENTRYPOINT ["/usr/bin/wget"]
  CMD ["https://cdn.wp.nginx.com/wp-content/uploads/2016/07/docker-swarm-hero2.png"]



.. code-block:: console

  docker run f9f41698e2f8 https://cdn-images-1.medium.com/max/1600/1*_NQN6_YnxS29m8vFzWYlEg.png


docker tag
-----------

To tag a local image with ID "e23aaea5dff1" into the "ubuntu_wget" image name repository with version "1.0":

.. code-block:: console

  docker tag e23aaea5dff1 ubuntu_wget:1.0


Build exercise
--------------

* Random numbers

* Copy the following short bash script in a file called random_numbers.bash.

.. code-block:: console

  #!/usr/bin/bash
  seq 1 1000 | shuf | head -$1


This script outputs random intergers from 1 to 1000: the number of integers selected is given as the first argument.

* Write a recipe for an image:

  * Based on centos:7

  * That will execute this script (with bash) when it is run, giving it 2 as a default argument (i.e. outputs 2 random integers): the default can be changed as the image is run.

  * Build the image.

  * Start a container with the default argument, then try it with another argument.

.. raw:: html

  <details>
  <summary><a>Suggested solution</a></summary>

.. code-block::

  FROM centos:7

  # Copy script from host to image
  COPY random_numbers.bash .

  # Make script executable
  RUN chmod +x random_numbers.bash

  # As the container starts, "random_numbers.bash" is run
  ENTRYPOINT ["/usr/bin/bash", "random_numbers.bash"]

  # default argument (that can be changed on the command line)
  CMD ["2"]

Build and run:

.. code-block:: console

  docker build -f Dockerfile_RN -t random_numbers .
  docker run random_numbers
  docker run random_numbers 10

.. raw:: html

  </details>

