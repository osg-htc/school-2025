---
status: testing
---

<style type="text/css"> pre em { font-style: normal; background-color: yellow; } pre strong { font-style: normal; font-weight: bold; color: \#008; } </style>

HTC Exercise 1.4: Read and Interpret Log Files
=================================================

## Exercise Goal

In the previous exercise, we learned how to translate a simple list of computational tasks into HTCondor jobs. What if we want to learn more about our jobs?

The goal of this exercise is to learn how to understand the contents of a job's log file, which contains a history of the steps HTCondor took to run your job. The log file is also a great place to look while you are testing your jobs, as it records resource usage.

Additionally, if you suspect something has gone wrong with your job, the log is the a great place to start looking for indications of whether things might have gone wrong (in addition to the error file).

## Reading a Log File

In our last exercise, we collected information about the slots and submitted a relatively small batch of jobs as a initial test. What is HTCondor doing behind the scenes? In addition, how many resources are we actually using?

Why should we care about our resource usage? There are two undesirable scenarios:

* **Under-requesting resources.** If you under-request resources (i.e. memory, disk), your jobs go into the hold state when their usage exceeds the resources allocated to it. This means your jobs stops running, and you have to fix the issue by requesting more resources and resubmit the jobs.
* **Over-requesting resources.** The easy solution to avoid the above scenario is to request a lot of resources, right? Unfortunately, no! Over-requesting resources means HTCondor needs to find a slot that has those resources, which can take longer than necessary if your jobs could have run on a slot with fewer resources. This is especially detrimental when you plan to submit many jobs.

For this exercise, we can examine a log file for any previous jobs that you have run. The example output below is based on a single job (process) within with the batch of jobs we submitted.

A job log file is updated throughout the life of a job, usually at key events. Each event starts with a heading that indicates what happened and when. Here are **some** of the event headings from the `tz_slotinfo` job log (detailed output in between headings has been omitted here):

``` file
000 (12636880.000.000) 2025-05-27 17:35:28 Job submitted from host: <128...
040 (12636880.000.000) 2025-05-27 17:35:52 Started transferring input files
040 (12636880.000.000) 2025-05-27 17:35:52 Finished transferring input files
021 (12636880.000.000) 2025-05-27 17:35:54 Message from starter on slot1...
001 (12636880.000.000) 2025-05-27 17:35:54 Job executing on host: <10.11...
006 (12636880.000.000) 2025-05-27 17:35:55 Image size of job updated: 1
040 (12636880.000.000) 2025-05-27 17:35:55 Started transferring output files
040 (12636880.000.000) 2025-05-27 17:35:55 Finished transferring output files
005 (12636880.000.000) 2025-05-27 17:35:55 Job terminated.
```

View one of these log files and scroll through, observing what's written. There is a lot of extra information in those lines, but you can see:

-   The job ID: cluster `12636880`, process `0` (written `000`)
-   The date and local time of each event
-   A brief description of the event: submission, execution, some information updates, and termination
-   Each event ends with a line that contains only 3 dots: `...`

!!! note
    Because we printed a single log file for all 50 jobs in the batch of jobs, all 50 jobs' events are printed in this log file as they happen. If you want an individual log file for each job in the batch, use `$(Process)` in the `log` line of the submit file.

However, some lines have additional information to help you quickly understand where and how your jobs are running. For example:  

``` file
001 (12636880.000.000) 2025-05-27 17:35:54 Job executing on host: <10.118.5.219:33393?CCBID=128.105.82.148:9618%3faddrs%3d128.105.82.148-9618+[2607-f388-2200-87-d439-a1c8-2a11-24fc]-9618%26alias%3dospool-ccb.osg.chtc.io%26noUDP%26sock%3dcollector1#72672458%20192.170.231.11:9618%3faddrs%3d192.170.231.11-9618+[fd85-ee78-d8a6-8607--1-73ab]-9618%26alias%3dospool-ccb.osgprod.tempest.chtc.io%26noUDP%26sock%3dcollector7#31809902&PrivNet=c219.mgmt.hellbender&addrs=10.118.5.219-33393&alias=c219.mgmt.hellbender&noUDP>
        SlotName: slot1_2@glidein_4082265_33522666@c219.mgmt.hellbender
        CondorScratchDir = "/local/scratch/glide_bKAhkg/execute/dir_1368654"
        Cpus = 1
        Disk = 1049600
        GLIDEIN_ResourceName = "Missouri-Hellbender-CE1"
        GPUs = 0
        Memory = 1024
...
```
-   The `SlotName` is the name of the execution point slot your job was assigned to by HTCondor, and the name of the execution point resource is provided in `GLIDEIN_ResourceName`.
-   The `CondorScratchDir` is the name of the scratch directory that was created by HTCondor for your job to run inside.
-   The `Cpu`, `GPUs`, `Disk` (in KiB), and `Memory` (in MB) values provide the maximum amount of each resource your job can use while running.

Another example of is the periodic update:

``` file
006 (12636880.000.000) 2025-05-27 17:35:55 Image size of job updated: 1
        0  -  MemoryUsage of job (MB)
        0  -  ResidentSetSize of job (KB)
...
```

These updates record the amount of memory that your jobs are using on the Execution Points. This can be helpful information, so that in future runs of the job, you can tell HTCondor how much memory you will need.

The job termination event includes a lot of very useful information:

``` file
005 (12636880.000.000) 2025-05-27 17:35:55 Job terminated.
        (1) Normal termination (return value 0)
                Usr 0 00:00:00, Sys 0 00:00:00  -  Run Remote Usage
                Usr 0 00:00:00, Sys 0 00:00:00  -  Run Local Usage
                Usr 0 00:00:00, Sys 0 00:00:00  -  Total Remote Usage
                Usr 0 00:00:00, Sys 0 00:00:00  -  Total Local Usage
        147  -  Run Bytes Sent By Job
        211  -  Run Bytes Received By Job
        147  -  Total Bytes Sent By Job
        211  -  Total Bytes Received By Job
        Partitionable Resources :    Usage  Request Allocated
           Cpus                 :        0        1         1
           Disk (KB)            :      130  1048576   1049600
           GPUs                 :                           0
           Memory (MB)          :        0     1024      1024
           TimeExecute (s)      :        1
           TimeSlotBusy (s)     :        3
...
```

Probably the most interesting information is:

-   The `return value` or `exit code` (`0` here, means the executable completed and didn't indicate any internal errors; non-zero usually means failure)
-   The total number of bytes transferred each way, which could be useful if your network is slow
-   The `Partitionable Resources` table, especially disk and memory usage, which will inform larger submissions.

There are many other kinds of events, but the ones above will occur in almost every job log.

**Questions to consider**

* Did we under- or over-request resources for our jobs?
* Why do you think the CPU usage shows `0` instead of `1`?
* What might account for the difference between `TimeExecute` and `TimeSlotBusy`?

Discuss your answers to these questions with a neighbor or staff member.

## Understanding How HTCondor Writes Files

When HTCondor writes the output, error, and log files, does it erase the previous contents of the file or does it add new lines onto the end? Let’s find out!

For this exercise, we will use the `tz_slotinfo` job from earlier.

1.  Edit the submit file so it submits 5 jobs instead of 50.
1.  Submit the job three separate times in a row.
1.  Wait for all the jobs to finish.
1.  Examine the output file: Did HTCondor erase the previous contents for each job, or add new lines?
1.  Examine the log file carefully: What happened there? Pay close attention to the times and job IDs of the events.
1.  How can you modify the submit file so it creates a unique `.out` and `.err` file for every `condor_submit` attempt? 

For further clarification about how HTCondor handles these files, reach out to your neighbor or one of the other School staff.
