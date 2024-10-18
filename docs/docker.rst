.. _docker-page:

*******************
Docker
*******************

Introduction to Docker
========================


What is Docker?
-------------------

* Platform for developing, shipping and running applications.
* Infrastructure as application / code.
* First version: 2013.
* Company: originally dotCloud (2010), later named Docker.
* Established `Open Container Initiative <https://www.opencontainers.org/>`__.

As a software:

* `Docker Community Edition <https://www.docker.com/products/container-runtime>`__.
* Docker Enterprise Edition.

There is an increasing number of alternative container technologies and providers. Many of them are based on software components originally from the Docker stack, and they usually try to address some specific use cases or weak points. As an example, **Singularity**, which we introduce later in this course, is focused on HPC environments. 

Another case, **Podman** <https://podman.io> keeps high functional compatibility with Docker but with a different focus on technology (not keeping a daemon) and permissions. **We will use Podman in this course**.
``docker`` command will be an alias of ``podman`` in our environment.

Docker components
--------------------

.. image:: http://apachebooster.com/kb/wp-content/uploads/2017/09/docker-architecture.png
  :width: 700

* Read-only templates.
* Containers are run from them.
* Images are not run.
* Images have several layers.

Images versus containers
----------------------------

* **Image**: A set of layers, read-only templates, inert.
* An instance of an image is called a **container**.

When you start an image, you have a running container of this image. You can have many running containers of the same image.

*"The image is the recipe, the container is the cake; you can make as many cakes as you like with a given recipe."*

https://stackoverflow.com/questions/23735149/what-is-the-difference-between-a-docker-image-and-a-container

Podman setup
--------------

Place the following bit of code in ``~/.config/containers/storage.conf``:

.. code-block:: console

  [storage]
    driver = "overlay"
    graphroot = "/tmp/podman/$USER/.local/share/containers/storage"
    [storage.options]
      mount_program = "/usr/bin/fuse-overlayfs"
      ignore_chown_errors = "true"


Docker vocabulary
----------------------------

.. code-block:: console

  docker


.. image:: images/docker_vocab.png
  :width: 550

Get help:

.. code-block:: console

  docker run --help


.. image:: images/docker_run_help.png
  :width: 550


Using existing images
---------------------

Explore Docker hub
******************

Images can be stored locally or shared in a registry.


`Docker hub <https://hub.docker.com/>`__ is the main public registry for Docker images.


Let's search the keyword **ubuntu**:

.. image:: images/dockerhub_ubuntu.png
  :width: 900

docker pull: import image
*************************

* get latest image / latest release

.. code-block:: console

  docker pull ubuntu


.. image:: images/docker_pull.png
  :width: 650

* choose the version of Ubuntu you are fetching: check the different tags


.. code-block:: console

  docker pull ubuntu:22.04


Biocontainers
*************

https://biocontainers.pro/

Specific directory of Bioinformatics related entries

* Entries in `Docker hub <https://hub.docker.com/u/biocontainers>`__ and/or `Quay.io <https://quay.io>`__ (RedHat registry)

* Normally created from `Bioconda <https://bioconda.github.io>`__

Example: **FastQC**

https://biocontainers.pro/#/tools/fastqc


.. code-block:: console

    docker pull biocontainers/fastqc:v0.11.9_cv7

docker images: list images
--------------------------

.. code-block:: console

  docker images

.. image:: images/docker_images_list.png
  :width: 650

Each image has a unique **IMAGE ID**.

docker run: run image, i.e. start a container
---------------------------------------------

Now we want to use what is **inside** the image.


**docker run** creates a fresh container (active instance of the image) from a **Docker (static) image**, and runs it.


The format is:

docker run image:tag **command**

.. code-block:: console

  docker run ubuntu:22.04 /bin/ls


.. image:: images/docker_run_ls.png
  :width: 200

Now execute **ls** in your current working directory: is the result the same?


You can execute any program/command that is stored inside the image:

.. code-block:: console

  docker run ubuntu:22.04 /bin/whoami
  docker run ubuntu:22.04 cat /etc/issue


You can either execute programs in the image from the command line (see above) or **execute a container interactively**, i.e. **"enter"** the container.

.. code-block:: console

  docker run -it ubuntu:22.04 /bin/bash


Run container as daemon (in background)

.. code-block:: console

  docker run --detach ubuntu:22.04 tail -f /dev/null

Run container as daemon (in background) with a given name

.. code-block:: console

  docker run --detach --name myubuntu ubuntu:22.04 tail -f /dev/null


docker ps: check containers status
----------------------------------

List running containers:

.. code-block:: console

  docker ps


List all containers (whether they are running or not):

.. code-block:: console

  docker ps -a


Each container has a unique ID.

docker exec: execute process in running container
-------------------------------------------------

.. code-block:: console

  docker exec myubuntu uname -a


* Interactively

.. code-block:: console

  docker exec -it myubuntu /bin/bash

docker rm, docker rmi: clean up!
--------------------------------

.. code-block:: console

  docker rm myubuntu
  docker rm -f myubuntu


.. code-block:: console

  docker rmi ubuntu:22.04


Major clean
***********

Check used space

.. code-block:: console

  docker system df


Remove unused containers (and others) - **DO WITH CARE**

.. code-block:: console

  docker system prune


Remove ALL non-running containers, images, etc. - **DO WITH MUCH MORE CARE!!!**

.. code-block:: console

  docker system prune -a

* Reference: https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes


Volumes
=======

Docker containers are fully isolated. It is necessary to mount volumes in order to handle input/output files.

Syntax: **\--volume/-v** *host:container*

.. code-block:: console

  mkdir test
  touch test/test
  docker run --detach --volume $(pwd)/test:/scratch --name fastqc_container biocontainers/fastqc:v0.11.9_cv7 tail -f /dev/null
  docker exec -ti fastqc_container /bin/bash
  > ls -l /scratch
  > exit
