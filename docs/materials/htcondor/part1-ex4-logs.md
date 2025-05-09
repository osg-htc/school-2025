---
status: testing
---

<style type="text/css"> pre em { font-style: normal; background-color: yellow; } pre strong { font-style: normal; font-weight: bold; color: \#008; } </style>

HTC Exercise 1.4: Read and Interpret Log Files
=================================================

Exercise Goal
-----------------

The goal of this exercise is to learn how to understand the contents of a job's log file, which contains a "history" of the steps HTCondor took to run your job.
If you suspect something has gone wrong with your job, the log is the a great place to start looking for indications of whether things might have gone wrong (in addition to the .err file).

Additionally, the log file is a great place to look while you are testing your jobs, as it records resource usage.

This exercise is short, but you'll want to at least read over it before moving on.

Reading a Log File
------------------

For this exercise, we can examine a log file for any previous job that you have run. The example output below is based on the `sleep 60` job.

A job log file is updated throughout the life of a job, usually at key events. Each event starts with a heading that indicates what happened and when. Here are **all** of the event headings from the `sleep` job log (detailed output in between headings has been omitted here):

``` file
000 (5739.000.000) 2025-04-28 10:44:20 Job submitted from host: <128.104.100.43:9618?addrs=...>
040 (5739.000.000) 2025-04-28 10:45:10 Started transferring input files
040 (5739.000.000) 2025-04-28 10:45:10 Finished transferring input files
001 (5739.000.000) 2025-04-28 10:45:11 Job executing on host: <128.104.55.42:9618?addrs=...>
006 (5739.000.000) 2025-04-28 10:45:20 Image size of job updated: 72
040 (5739.000.000) 2025-04-28 10:45:20 Started transferring output files
040 (5739.000.000) 2025-04-28 10:45:20 Finished transferring output files
006 (5739.000.000) 2025-04-28 10:46:11 Image size of job updated: 4072
005 (5739.000.000) 2025-04-28 10:46:11 Job terminated.
```

There is a lot of extra information in those lines, but you can see:

-   The job ID: cluster 5739, process 0 (written `000`)
-   The date and local time of each event
-   A brief description of the event: submission, execution, some information updates, and termination

!!! note
    Each event ends with a line that contains only 3 dots: `...`

However, some lines have additional information to help you quickly understand where and how your job is running. For example:  

``` file
001 (5739.000.000) 2025-04-28 10:45:11 Job executing on host: <128.105.69.202:35973?CCBID=128.105.82.148:9618%3faddrs%3d128.105.82.148-9618+[2607-f388-2200-84-2188-4cc2-94a8-c303]-9618%26alias%3dospool-ccb.osg.chtc.io%26noUDP%26sock%3dcollector9#46060696%20192.170.231.11:9618%3faddrs%3d192.170.231.11-9618+[fd85-ee78-d8a6-8607--1-7398]-9618%26alias%3dospool-ccb.osgprod.tempest.chtc.io%26noUDP%26sock%3dcollector7#26736031&PrivNet=e4033.chtc.wisc.edu&addrs=128.105.69.202-35973+[2607-f388-2200-100-1270-fdff-fe56-c3c4]-35973&alias=e4033.chtc.wisc.edu&noUDP>
        SlotName: slot1_5@e4033.chtc.wisc.edu
        CondorScratchDir = "/var/lib/condor/execute/slot1/dir_3037242/glide_LLniGP/execute/dir_378707"
        Cpus = 1
        Disk = 1049643
        GLIDEIN_ResourceName = "GLOW"
        GPUs = 0
        Memory = 1024
...
```
-   The `SlotName` is the name of the execution point slot your job was assigned to by HTCondor, and the name of the execution point resource is provided in `GLIDEIN_ResourceName`.
-   The `CondorScratchDir` is the name of the scratch directory that was created by HTCondor for your job to run inside.
-   The `Cpu`, `GPUs`, `Disk`, and `Memory` values provide the maximum amount of each resource your job can use while running.

Another example of is the periodic update:

``` file
006 (5739.000.000) 2025-04-28 10:45:20 Image size of job updated: 72
    1  -  MemoryUsage of job (MB)
    72  -  ResidentSetSize of job (KB)
...
```

These updates record the amount of memory that the job is using on the execute machine. This can be helpful information, so that in future runs of the job, you can tell HTCondor how much memory you will need.

The job termination event includes a lot of very useful information:

``` file
005 (5739.000.000) 2025-04-28 10:46:11 Job terminated.
        (1) Normal termination (return value 0)
                Usr 0 00:00:00, Sys 0 00:00:00  -  Run Remote Usage
                Usr 0 00:00:00, Sys 0 00:00:00  -  Run Local Usage
                Usr 0 00:00:00, Sys 0 00:00:00  -  Total Remote Usage
                Usr 0 00:00:00, Sys 0 00:00:00  -  Total Local Usage
        306  -  Run Bytes Sent By Job
        683  -  Run Bytes Received By Job
        306  -  Total Bytes Sent By Job
        683  -  Total Bytes Received By Job
        Partitionable Resources :    Usage  Request Allocated
           Cpus                 :        0        1         1
           Disk (KB)            :      139  1048576   1049858
           GPUs                 :                           0
           Memory (MB)          :        4     1024      1024
           TimeExecute (s)      :       12
           TimeSlotBusy (s)     :       14

        Job terminated of its own accord at 2025-05-01T15:59:27Z with exit-code 0.
...
```

Probably the most interesting information is:

-   The `return value` or `exit code` (`0` here, means the executable completed and didn't indicate any internal errors; non-zero usually means failure)
-   The total number of bytes transferred each way, which could be useful if your network is slow
-   The `Partitionable Resources` table, especially disk and memory usage, which will inform larger submissions.

There are many other kinds of events, but the ones above will occur in almost every job log.


Understanding When Job Log Events Are Written
---------------------------------------------

When are events written to the job log file? Let’s find out. Read through the entire procedure below before starting, because some parts of the process are time sensitive.

1.  Change the `sleep` job submit file, so that the job sleeps for 2 minutes (= 120 seconds)
1.  Submit the updated sleep job
1.  As soon as the `condor_submit` command finishes, hit the return key a few times, to create some blank lines
1.  Right away, run a command to show the log file and **keep showing** updates as they occur:

        :::console
        [username@ap40]$ tail -f sleep.log

1.  Watch the output carefully. When do events appear in the log file?
1.  After the termination event appears, press CTRL+C to end the `tail` command and return to the shell prompt.


Understanding How HTCondor Writes Files
---------------------------------------

When HTCondor writes the output, error, and log files, does it erase the previous contents of the file or does it add new lines onto the end? Let’s find out!

For this exercise, we can use the `hostname` job from earlier.

1.  Edit the `hostname` submit file so that it uses new and unique filenames for output, error, and log files.  
Alternatively, delete any existing output, error, and log files from previous runs of the `hostname` job.
1.  Submit the job three separate times in a row (there are better ways to do this, which we will cover in the next lecture)
1.  Wait for all three jobs to finish
1.  Examine the output file: How many hostnames are there? Did HTCondor erase the previous contents for each job, or add new lines?
1.  Examine the log file… carefully: What happened there? Pay close attention to the times and job IDs of the events.

For further clarification about how HTCondor handles these files, reach out to your mentor or one of the other school staff.
