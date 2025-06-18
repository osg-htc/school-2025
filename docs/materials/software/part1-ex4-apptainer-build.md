---
status: testing
---

<style type="text/css">
  pre em { font-style: normal; background-color: yellow; }
  pre strong { font-style: normal; font-weight: bold; color: \#008; }
</style>

Software Exercise 1.4: Build, Test, and Deploy an Apptainer Container
====================================

**Objective**: to practice building and using a custom
apptainer container

**Why learn this?**: You may need to go through this process if you 
want to use a container for your jobs and can't find one that has 
what you need. 

Motivating Script
-----------------

1. Create a script called `hello-cow.py`:

		:::file
		#!/usr/bin/env python3

		import cowsay
		cowsay.cow('Hello OSG User School')

1. Give it executable permissions: 

		:::console
		$ chmod +x hello-cow.py

1. Try running the script:

		:::console
		$ ./hello-cow.py

	It will likely fail, because the cowsay library isn't installed. This is a 
	scenario where we will want to build our own container that includes a base 
	Python installation and the `cowsay` Python library. 

Preparing a Definition File
---------------------------

We can describe our desired Apptainer image in a special format called a 
**definition file**. This has special keywords that will direct Apptainer 
when it builds the container image. 

1. Create a file called `py-cowsay.def` with these contents: 

		:::file
		Bootstrap: docker
		From: hub.opensciencegrid.org/htc/ubuntu:22.04

		%post
			apt-get update -y
			apt-get install -y \
					python3-pip \
					python3-numpy
			python3 -m pip install cowsay

Note that we are starting with the same `ubuntu` base we used in previous 
exercises. The `%post` statement includes our installation commands, including 
updating the `pip` and `numpy` packages, and then using `pip` to install `cowsay`.

To learn more about definition files, see [Exercise 3.1](part3-ex1-apptainer-recipes.md)

Build the Container
-------------------

Once the definition file is complete, we can build the container. 

!!! warning "Ensure environment is ready for Apptainer commands"
	If your login has been interrupted or you've changed terminals since
	[Software Exercise 1.1](/school-2025/materials/software/part1-ex1-run-apptainer), 
	then make sure to run the commands in the
	[Setup section](/school-2025/materials/software/part1-ex1-run-apptainer/#setup)
	of that exercise before proceeding!

1. Run the following command to build the container: 

		:::console
		$ apptainer build py-cowsay.sif py-cowsay.def

As with the Docker image in the [previous exercise](part1-ex3-docker-jobs.md), 
the first argument is the name to give to the newly create image file and the 
second argument is how to build the container image - in this case, the definition file. 


Testing the Image Locally
-------------------

1. Do you remember how to interactively test an image? Look back 
at [Exercise 1.1](part1-ex1-run-apptainer.md) and guess what command would 
allow us to test our new container. 

1. Try running: 

		:::console
		$ apptainer shell py-cowsay.sif

1. Then try running the `hello-cow.py` script: 

		:::console
		Apptainer> ./hello-cow.py

1. If it produces an output, our container works! We can now exit (by typing `exit`)
and submit a job. 

Submit a Job
--------------

1. Make a copy of a submit file from a previous exercise in this section. Can you 
guess what options need to be used or modified? 

1. Make sure you have the following (in addition to `log`, `error`, `output` and 
CPU and memory requests): 

		:::file
		universe = container
		container_image = py-cowsay.sif
		
		executable = hello-cow.py

1. Submit the job and verify the output when it completes. 

		:::file
		  ______________________
		| Hello OSG User School! |
		  ======================
							  \
							   \
								 ^__^
								 (oo)\_______
								 (__)\       )\/\
									 ||----w |
									 ||     ||

!!! warning "Proper location for container `.sif` files"
	The above example is storing the `py-cowsay.sif` file in the home directory on the Access Point,
	and in turn that means the job will transfer the file via the Access Point. Container image files,
	however, are typically large and if you are submitting many jobs, the Access Point can be overwhelmed
	trying to transfer so many large files!

	In practice, container image files like this one should be placed in your `$DATA` directory, and the
	submit file should use the `osdf:///` protocol to declare the transfer. For more information, see the
	[Data Exercises](/school-2025/materials/data/part2-ex1-osdf-inputs/) or the 
	[OSPool guide on using the OSDF](https://portal.osg-htc.org/documentation/htc_workloads/managing_data/osdf/).

Apply to Your Work
------------------

1. Have you ever wanted to "just install" your software for use on the OSPool? 
   Could you accomplish that by building your own Apptainer container?

1. Do you know how to install your software on a brand new computer?

    - If so, how you would incorporate those instructions into an Apptainer definition file?
    - If not, can you find the necessary instructions?

1. Do you have a simple test you can use to check if the software you want to use is working as expected?

