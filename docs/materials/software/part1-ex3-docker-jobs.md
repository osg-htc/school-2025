---
status: testing
---

<style type="text/css"> pre em { font-style: normal; background-color: yellow; } pre strong { font-style: normal; font-weight: bold; color: #008; } </style>

Software Exercise 1.3: Use Docker Containers in OSPool Jobs
====================================

**Objective**: Create a local copy of a Docker container, use it to submit a job. 

**Why learn this?**: Same as the previous exercise; this may also be how you end up 
submitting your jobs if you can find an existing Docker container with your software.  

Create Local Copy of Docker Container
-------------------

While it is technically possible to use a Docker container directly in a job, 
there are some good reasons for converting it to a local Apptainer container first. 
We'll do this with the same `python:3.10` Docker container we used in the 
[first exercise](part1-ex1-run-apptainer.md). 

!!! warning "Ensure environment is ready for Apptainer commands"
	If your login has been interrupted or you've changed terminals since
	[Software Exercise 1.1](/school-2025/materials/software/part1-ex1-run-apptainer), 
	then make sure to run the commands in the
	[Setup section](/school-2025/materials/software/part1-ex1-run-apptainer/#setup)
	of that exercise before proceeding!

To convert the Docker container to a local Apptainer container, run: 

	:::console
	$ apptainer build local-py310.sif docker://python:3.10

The first argument after `build` is the name of the new Apptainer container file, the 
second argument is what we're building from (in this case, Docker). 

Submit File and Executable
-------------------

1.  Make a copy of your submit file from the [previous container exercise](part1-ex2-apptainer-jobs.md) or build from an existing submit file. 

1.  Add the following lines to the submit file or modify existing lines to match the lines below: 

		:::file
		universe = container
		container_image = local-py310.sif

1.  Use the same executable as the [previous exercise](part1-ex2-apptainer-jobs.md). 

1. Once these steps are done, submit the job. You might get a warning about using OSDF for container transfers - ignore this warning for now.

!!! warning "Proper location for container `.sif` files"
	The above example is storing the `local-py310.sif` file in the home directory on the Access Point,
	and in turn that means the job will transfer the file via the Access Point. Container image files,
	however, are typically large and if you are submitting many jobs, the Access Point can be overwhelmed
	trying to transfer so many large files!

	In practice, container image files like this one should be placed in your `$DATA` directory, and the
	submit file should use the `osdf:///` protocol to declare the transfer. For more information, see the
	[Data Exercises](/school-2025/materials/data/part2-ex1-osdf-inputs/) or the 
	[OSPool guide on using the OSDF](https://portal.osg-htc.org/documentation/htc_workloads/managing_data/osdf/).

Finding Docker Containers
-------------

There are a lot of Docker containers on Docker Hub, but they are not all 
created equal. Anyone can create an account on Docker Hub and share container images there, so it’s important to exercise caution when choosing a container image on Docker Hub. These are some indicators that a container image on Docker Hub is consistently maintained, functional and secure:

- The container image is updated regularly.
- The container image is associated with a well established company, community, or other group that is well-known.
- There is a Dockerfile or other listing of what has been installed to the container image.
- The container image page has documentation on how to use the container image. [^1]

Given these indicators:

1. Can you find a container on [Docker Hub](https://hub.docker.com/) that would be 
useful for running Jupyter notebooks that use tensorflow? 

1. Does your chosen image meet at least 2 of the criteria above? 

Apply to Your Work
------------------

1. Do you or your colleagues use Docker containers for running calculations?

1. Is the software you want to use already available via a container registry 
   such as [DockerHub](https://hub.docker.com/), [NVIDIA Catalog](https://catalog.ngc.nvidia.com/containers), or elsewhere?

[^1]: This list and previous text taken from [Introduction to Docker](https://carpentries-incubator.github.io/docker-introduction/)
