---
status: ready
---

<style type="text/css"> pre em { font-style: normal; background-color: yellow; } pre strong { font-style: normal; font-weight: bold; color: \#008; } </style>

Software Exercise 1.1: Run and Explore Containers
============================================================

**Objective**: Run a container interactively

**Why learn this?**: Being able to run a container directly allows you to confirm 
what is installed and whether any additional scripts or code will work in the context 
of the container. 

Setup
--------

Make sure you are logged into `ap40.uw.osg-htc.org`.  For this exercise 
we will be using Apptainer containers maintained by OSG staff or existing 
containers on Docker Hub. 

There is some set-up that we should do to help lighten the load on the 
Access Point as we work with containers. 

First, download the `apptainer-setup.sh` script:

```
osdf object get /ospool/uc-shared/public/school/2025/dev/apptainer-setup.sh ./
```

Then run the script using this command:

```
. apptainer-setup.sh
```

You should see a message that the setup has been completed.
(This is the only time you'll need to run this script.)

Exploring Apptainer Containers
-------------------

First, let's try to run a container from the [OSG-Supported List](https://portal.osg-htc.org/documentation/htc_workloads/using_software/available-containers-list/). 

1. Find the full path for the `ubuntu 22.04` container image. 

1. To run it, use this command: 

		:::console
		$ apptainer shell /cvmfs/singularity.opensciencegrid.org/htc/ubuntu:22.04

	It may take a few minutes to start - don't worry if this happens. 

	!!! warning "About the `/cvmfs` path"
		In the above example, we used a path beginning with `/cvmfs` to launch a container.
        
        **This should be the only place that you use such a path!**

1. Once the container starts, the prompt will change to either `Singularity>` or 
  `Apptainer>`. Run `ls` and `pwd`. Where are you? Do you see your files? 

1. The `apptainer shell` command will automatically connect your home directory to 
the running container so you can use your files. 

1. How do we know we're in a different Linux environment? Try printing out the Linux 
version, or checking the version of common tools like `gcc` or Python: 

		:::console
		$ grep "PRETTY_NAME" /etc/os-release 
		$ gcc --version
		$ python3 --version

	!!! note "Full Linux version information"
		In the above example, we used `grep` to only show the value of the `PRETTY_NAME`
		in the `/etc/os-release` file. There's a lot more information in that file
		about the Linux version, which you can see with 
		
			:::console
			$ cat /etc/os-release

1. Exit out of the container by typing `exit`. 

1. Type the same commands back on the normal Access Point. Should they give the same 
results as when typed in the container, or different? 

		:::console
		$ grep "PRETTY_NAME" /etc/os-release 
		$ gcc --version
		$ python3 --version

Exploring Docker Containers
------------------

The process for interactively running a Docker container will be very 
similar to an apptainer container. The main difference is a `docker://` prefix 
before the container's identifying name. 

1. We are going to be using a [Python image from Docker Hub](https://hub.docker.com/_/python). 
Click on the "Tags" tab to see all the different versions of this container that exists. 

1. Let's use version `3.10`. To run it interactively, use this command: 

		:::console
		$ apptainer shell docker://python:3.10

1. Once the container starts and the prompt changes, try running similar commands 
as above. What version of Linux is used in this container? Does the version of Python 
match what you expect, based on the name of the container? 

1. Once done, type `exit` to leave the container. 

Apply to Your Work
------------------

1. Is the software you want to use already available via a container registry 
   such as [DockerHub](https://hub.docker.com/), [NVIDIA Catalog](https://catalog.ngc.nvidia.com/containers), or elsewhere?

1. Consider what it takes to install your software - if you could choose the operating system, would that make it easier to install your software?

