---
status: testing
---

# OSPool Exercise 1.3: How Does Capacity Change?

The previous exercise said that OSPool capacity is constantly changing.
Let’s explore that!

This is one exercise that you **cannot** finish today.
The main idea is to capture some statistics about the OSPool today,
then capture the same statistics another day and compare.

## Capturing Some OSPool Statistics

Below are several commands to capture statistics about available capacity and its characteristics in the OSPool.
Each one saves its data to a text file.

Note the `sort | uniq -c` part of each command —
this reduces the output by combining identical lines of output and adding counts of identical lines.
The count comes first.
So, looking at the first few lines of saved output from the first (CPUs) command:

```
    874 1
    470 2
    186 3
```

there were 874 slots available with one CPU core,
470 slots with two CPU cores,
and 186 slots with three CPUs cores (and so on).

The steps are easy here:

1.  Make sure you are logged in to `ap40.uw.osg-htc.org`
1.  Create and change into a new folder for this exercise, for example `ospool-ex13`
1.  Run each of the commands below

    Number of available slots by CPUs available in that slot:

        :::console
        $ condor_status -avail -af CPUs | sort -n | uniq -c > cpus-snapshot1.txt

    Number of available slots by GPUs:

        :::console
        $ condor_status -avail -af GPUs | sort -n | uniq -c > gpus-snapshot1.txt

    Number of available slots by memory per CPU core, rounded up to the nearest 2 GB:

        :::console
        $ condor_status -avail -af "eval(quantize(Memory/CPUs, 2048))" | sort -n | uniq -c > memory-snapshot1.txt

    Number of available slots by operating system and version:

        :::console
        $ condor_status -avail -af OpSysAndVer | sort | uniq -c > opsysandver-snapshot1.txt

    Number of available slots by HTCondor version of the Execution Point:

        :::console
        $ condor_status -avail -af CondorVersion | sort | uniq -c > condorver-snapshot1.txt

    Number of available slots by Pelican plugin version on the Execution Point (which you will learn about Wednesday):

        :::console
        $ condor_status -avail -af PelicanPluginVersion | sort | uniq -c > pelicanver-snapshot1.txt

Of course, you are welcome to look at and think about the data you collected.
And as usual, talk to neighbors or staff about it, if you like!

Stop here and come back another day. (We’ll try to remind you.)

## Capturing New Statistics

Go back to the same exercise directory you used earlier, and run the same commands as before.

**However**, be sure to change the output filenames for each one, or you will overwrite your previous data.
Here are the commands again, with different output filenames:

        :::console
        $ condor_status -avail -af CPUs | sort -n | uniq -c > cpus-snapshot2.txt
        $ condor_status -avail -af GPUs | sort -n | uniq -c > gpus-snapshot2.txt
        $ condor_status -avail -af "eval(quantize(Memory/CPUs, 2048))" | sort -n | uniq -c > memory-snapshot2.txt
        $ condor_status -avail -af OpSysAndVer | sort | uniq -c > opsysandver-snapshot2.txt
        $ condor_status -avail -af CondorVersion | sort | uniq -c > condorver-snapshot2.txt
        $ condor_status -avail -af PelicanPluginVersion | sort | uniq -c > pelicanver-snapshot2.txt

Per type of file (cpus-, gpus-, etc.), compare your snapshots.
Do you see any interesting differences?
Some are more likely to change than others.
Once again, talk to others about what you found, if you like.
