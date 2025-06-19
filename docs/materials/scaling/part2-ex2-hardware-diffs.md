---
status: testing
---

# Scaling Up Exercise 2.2: Hardware Differences Between OSPool and the CHTC Pool

The goal of this exercise is to compare hardware differences between the 
Open Science Pool and a local HTC system (the CHTC pool), using similar 
tools as the Tuesday exercises. 

Specifically, we will look at how easy it is to get access to resources
in terms of the amount of memory that is requested.
This will not be a very careful study,
but should give you some idea of one way in which the pools are different.

In the first two parts of the exercise,
you will submit batches of jobs that differ only in how much memory each one requests.
We will request memory from 2 - 128 GB, doubling the memory each time.
One set of jobs will be submitted to CHTC, and the other, identical set of jobs will be submitted to the OSPool.
You will check the queue periodically to see how many jobs have completed and how many are still waiting to run.

## Checking CHTC memory availability

In this first part, you will create the submit file that will be used for both sets of 
jobs. 

!!! Tip 
	Make sure you are logged into `ap2003.chtc.wisc.edu` for the first part of this exercise. 

### Create the submit file

To create our parameter sweep,
we will create a **new** submit file with the queue...in syntax
and change the value of our parameter (`request_memory`) for each batch of jobs.

1. Creat a subdirectory for this exercise (we've used `mem-requests` below). 
1. In this folder, create this template submit file: 

		executable          = /usr/bin/sleep
		arguments           = 30
		transfer_executable = false
		
		log                 = memory_sweep.$(Cluster).log
		
		request_cpus        = 1
		request_disk 		= 1024
		
		queue 5 request_memory in (
		1GB
		2GB
		3GB
		)

1.  The queue statement, is iterating through the list shown between 
the parentheses (1, 2, 3) using those values as the memory request. Five 
jobs are submitted for each memory value. 
1. **Edit the list** to reflect different 
amounts of memory. We recommend this list in gigabytes (GB): 2, 4, 8, 16, 32, 64, 128, 256, 512
1.  Submit the submit file (should be about 30-45 jobs). 

### Monitoring the local jobs

Every few minutes, run `condor_q` and see how your sleep jobs are doing.
To display the number of jobs remaining for each `request_memory` parameter specified, 
run the following command:

``` console
$ condor_q <Cluster ID> -af RequestMemory | sort -n | uniq -c
```

The numbers in the left column are the number of jobs left of that type
and the number on the right is the amount of memory you requested, in MB.
Consider making a little table like the one below to track progress.

| Memory | Remaining \#1 | Remaining \#2 | Remaining \#3 |
|:-------|:--------------|:--------------|:--------------|
| 8 GB   | 10            | 6             |               |
| 16 GB  | 10            | 7             |               |
| 32 GB  | 10            | 8             |               |
| 64 GB  | 10            | 9             |               |

In the meantime, between checking on your local jobs, start the next section –
but take a break every few minutes to switch back to `CHTCSUBMITNODE` and record progress on your jobs.

## Checking OSPool memory availability

Now you will do essentially the same thing on the OSPool.

1.  Copy the `mem-requests` directory from the [section above](#checking-path-memory-availability)
    from `ap2003.chtc.wisc.edu` to `ap40.uw.osg-htc.org`. 

    ??? success "Not sure how to copy the files? "
    	There are different ways to copy files from server to server. For this 
    	task, one option is: 
    	
    	```
    	[user.name@ap2003 ~]$ scp -r path/to/mem-requests user.name@ap40.uw.osg-htc.org:/home/user.name
    	```
    	Edit the username and the paths to reflect your home directory organization. 

1.  Log in or switch to `ap40.uw.osg-htc.org`

1.  Submit the jobs to the OSPool

### Monitoring the OSPool jobs

As you did in the first part, use `condor_q` to track how your sleep jobs are doing.
It is fine to move on to the next exercise, but keep tracking the status of both sets of these jobs.
After you are done with the [next exercise](part2-ex2-software-diffs.md),
come back to this exercise and analyze the results.

## Analyzing the results

Have all of your jobs from this exercise completed on both CHTC and the OSPool?

* How many jobs have completed thus far on CHTC?
* How many have completed thus far on the OSPool?

Due to the dynamic nature of the OSPool,
the demand for higher memory jobs there may have resulted in a temporary increase in high-memory slots there.
That being said, high-memory are a high-demand, low-availability resource in the OSPool
so your 64&nbsp;GB jobs (or higher!) may have taken longer to run or complete.
On the other hand, CHTC has a fair number of 64&nbsp;GB (and greater) slots
so all your jobs have a high chance of running.
