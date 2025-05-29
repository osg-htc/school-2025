---
status: testing
---

<style type="text/css"> pre em { font-style: normal; background-color: yellow; } pre strong { font-style: normal; font-weight: bold; color: \#008; } </style>

HTC Exercise 1.2: Experiment With HTCondor Commands
===================================================

## Exercise Goal

The goal of this exercise is to learn about two HTCondor commands: `condor_q` and `condor_status`.
These commands are useful for monitoring your jobs and viewing available Execution Point slots throughout the week.

## Viewing Slots

As discussed in the lecture, the `condor_status` command is used to view the current state of slots in an HTCondor pool.

At its most basic, the command is:

``` console
[username@ap40 ~]$ condor_status
```

When running this command, there is typically a lot of output printed to the screen. Looking at your terminal output, there is one line per execute point slot. 

!!! tip

    You can widen your terminal window or zoom out, which may help you to see all details of the output better. Since there are so many entries, you can also limit the number of slots displayed with `condor_status -limit 5`.

Here is some example output (what you see will be longer):

``` console
Name                                                 OpSys      Arch   State     Activity LoadAv Mem   ActvtyTime

slot1_15@IU-Jetstream2-Backfill.e53d75d92d0e         LINUX      X86_64 Claimed   Busy      0.000 2048  0+00:00:08
slot1_7@IU-Jetstream2-Backfill.f7d8ef98fdd0          LINUX      X86_64 Claimed   Busy      1.000 8192  0+04:57:31
slot1@glidein_2_813658338@e4014.chtc.wisc.edu        LINUX      X86_64 Unclaimed Idle      0.000   14  0+05:35:29
slot1@glidein_608506_86504717@node063.lawrence       LINUX      X86_64 Unclaimed Idle      0.000  472  0+15:59:56
slot1_4@glidein_3651085_902238852@uct2-c576.mwt2.org LINUX      X86_64 Claimed   Busy      4.470 2048  0+05:10:48
```

This output consists of 8 columns:

| Column      | Example                      | Meaning                                                                                                                 |
|:-----------|:-----------------------------|:------------------------------------------------------------------------------------------------------------------------|
| Name       | `slot1_4@glidein_3651085_902238852@uct2-c576.mwt2.org` | Full slot name (including the hostname)                                                                                                  |
| OpSys      | `LINUX`                      | Operating system                                                                                                        |
| Arch       | `X86_64`                     | Slot architecture (e.g., Intel 64 bit)                                                                               |
| State      | `Claimed`                    | State of the slot (`Unclaimed` is available, `Owner` is being used by the machine owner, `Claimed` is matched to a job) |
| Activity   | `Busy`                       | Is there activity on the slot?                                                                                          |
| LoadAv     | `4.470`                      | Load average, a measure of CPU activity on the slot                                                                     |
| Mem        | `2048`                       | Memory available to the slot, in MB                                                                                     |
| ActvtyTime | `0+05:10:48`                 | Amount of time spent in current activity (days + hours:minutes:seconds)                                                 |

At the end of the slot listing, there is a summary. Here is an example:

``` console
               Total Owner Claimed Unclaimed Matched Preempting  Drain Backfill BkIdle

  X86_64/LINUX 54466     0   47991      6313       0        162      0        0      0
 aarch64/LINUX     1     0       0         1       0          0      0        0      0

         Total 54467     0   47991      6314       0        162      0        0      0
```

There is one row of summary for each machine (i.e. "slot") architecture/operating system combination with columns for the number of slots in each state. The final row gives a summary of slot states for the whole pool.

### Questions

-   When you run `condor_status`, how many 64-bit Linux slots are available? (Hint: Unclaimed = available.)
-   What percent of the total slots are currently claimed by researcher jobs? (Note: there is a rapid turnover of slots, which is what allows users with new submission to have jobs start quickly.)
-   How have these numbers changed (if at all) when you run the `condor_status` command again?

## Viewing Researcher Jobs

The `condor_q` command lists jobs that were submitted from the Access Point and are running or waiting to run. The `_q` part of the name is an abbreviation of “queue”, or the list of jobs *waiting* to finish.

### Viewing Your Own Jobs

The default behavior of the command lists only your jobs:

``` console
[username@ap40 ~]$ condor_q
```

The main part of the output (which will be empty, because you haven't submitted jobs yet) shows one set ("batch") of submitted jobs per line. If you had a single job in the queue, it might look something like the below:

``` console
-- Schedd: ap40.uw.osg-htc.org : <128.105.68.62:9618?... @ 04/28/25 13:54:57
OWNER  BATCH_NAME            SUBMITTED   DONE   RUN    IDLE  TOTAL JOB_IDS
alice CMD: run_ffmpeg.sh   4/28 09:58      _      _      1      1 18801.0               
```

This output consists of 9 columns:

| Column         | Example         | Meaning                                                                                                                        |
|:------------|:----------------|:-------------------------------------------------------------------------------------------------------------------------------|
| OWNER       | `alice`        | The user ID of the user who submitted the job                                                                                  |
| BATCH\_NAME | `run_ffmpeg.sh` | The "jobbatchname" specified within the submit file. If not specified, it's given the value of `ID: <ClusterID>` | 
| SUBMITTED   | `4/28 09:58`    | The date and time when the job was submitted                                                                                   |
| DONE        | `_`             | Number of jobs in this batch that have completed                                                                               |
| RUN         | `_`             | Number of jobs in this batch that are currently running                                                                        |
| IDLE        | `1`             | Number of jobs in this batch that are idle, waiting for a match                                                                |
| HOLD        | `_`             | Column will show up if there are jobs on "hold" because something about the submission/setup needs to be corrected by the user |
| TOTAL       | `1`             | Total number of jobs in this batch                                                                                             |
| JOB\_IDS    | `18801.0`       | Job ID or range of Job IDs in this batch with the format `<ClusterID>.<ProcessID>`                                                                                |

At the end of the job listing, there is a summary. Here is a sample:

``` console
1 jobs; 0 completed, 0 removed, 1 idle, 0 running, 0 held, 0 suspended
```

It shows total counts of jobs in the different possible states.

**Questions:**

-   For the sample above, when was the job submitted?
-   For the sample above, was the job running or not yet? How can you tell?

### Viewing Everyone’s Jobs

By default, the `condor_q` command shows **your** jobs only. To see everyone’s jobs that are queued on the machine, add the `-all` option:

``` console
[username@ap40 ~]$ condor_q -all
```

-   How many jobs are queued in total (i.e., running or waiting to run)?
-   How many jobs from this submit machine are running right now?

### Viewing Jobs without the Default "batch" Mode

The `condor_q` output, by default, groups "batches" of jobs together (i.e. if they were submitted with the same submit file or "jobbatchname"). To see more information for EVERY job on a separate line of output, use the `-nobatch` option to `condor_q`:

``` console
[username@ap40 ~]$ condor_q -all -nobatch
```

**How has the column information changed?** (Below is an example of the top of the output.)

``` console
-- Schedd: ap40.uw.osg-htc.org : <128.105.68.62:9618?... @ 04/28/25 13:57:40
 ID       OWNER            SUBMITTED     RUN_TIME ST PRI SIZE   CMD
18203.0   s16_alirezakho  4/28 09:51   0+00:00:00 I  0      0.7 pascal
18204.0   s16_alirezakho  4/28 09:51   0+00:00:00 I  0      0.7 pascal
18801.0   alice           4/28 09:58   0+00:00:00 I  0      0.0 run_ffmpeg.sh
18997.0   s16_martincum   4/28 10:59   0+00:00:32 R  0    733.0 runR.pl 1_0 run_perm.R 1 0 10
19027.5   s16_martincum   4/28 11:06   0+00:09:20 R  0   2198.0 runR.pl 1_5 run_perm.R 1 5 1000
```

The `-nobatch` output shows a line for every job and consists of 8 columns:

| Col       | Example         | Meaning                                                                        |
|:----------|:----------------|:-------------------------------------------------------------------------------|
| ID        | `18801.0`       | Job ID, which is the `cluster`, a dot character (`.`), and the `process`       |
| OWNER     | `alice`         | The user ID of the user who submitted the job                                  |
| SUBMITTED | `4/28 09:58`    | The date and time when the job was submitted                                   |
| RUN\_TIME | `0+00:00:00`    | Total time spent running so far (days + hours:minutes:seconds)                 |
| ST        | `I`             | Status of job: `I` is Idle (waiting to run), `R` is Running, `H` is Held, etc. |
| PRI       | `0`             | Job priority (see next lecture)                                                |
| SIZE      | `0.0`           | Current run-time memory usage, in MB                                           |
| CMD       | `run_ffmpeg.sh` | The executable command (with arguments) to be run                              |

**In future exercises, you'll want to switch between `condor_q` and `condor_q -nobatch` to see different types of information about YOUR jobs.**

Extra Information
-----------------

Both `condor_status` and `condor_q` have many command-line options, some of which significantly change their output.
You will explore a few of the most useful options in future exercises, but if you want to experiment now, go ahead!
There are a few ways to learn more about the commands:

-   Use the (brief) built-in help for the commands, e.g.: `condor_q -h`.
-   Read the installed man(ual) pages for the commands, e.g.: `man condor_q`
-   Find the command in [the online manual](https://htcondor.readthedocs.io/en/latest/); **note:** the text online is the same as the `man` text, only formatted for the web

As a simple exercise, use one of the above reference methods to learn about the `-limit` option and implement it in a `condor_q` command.
