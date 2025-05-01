---
status: in progress
---

<style type="text/css"> pre em { font-style: normal; background-color: yellow; } pre strong { font-style: normal; font-weight: bold; color: \#008; } </style>

Bonus HTC Exercise 1.9: Explore condor_status
===========================================

The goal of this exercise is try out some of the most common options to the `condor_status` command, so that you can view slots effectively.

The main part of this exercise should take just a few minutes, but if you have more time later, come back and work on the extension ideas at the end to become a `condor_status` expert!

Selecting Slots
---------------

The `condor_status` program has many options for selecting which slots are listed. You've already learned the basic `condor_status` and the `condor_status -compact` variation (which you may wish to retry now, before proceeding).

Another convenient option is to list only those slots that are available now:

``` console
[username@ap40]$ condor_status -avail
```

Of course, the individual execute machines only report their slots to the collector at certain time intervals, so this list will not reflect the up-to-the-second reality of all slots. But this limitation is true of all `condor_status` output, not just with the `-avail` option.

Similar to `condor_q`, you can limit the slots that are listed in two easy ways. To list just the slots on a specific machine:

``` console
[username@ap40]$ condor_status <hostname>
```

For example, if you want to see the slots on `e2337.chtc.wisc.edu` (in the CHTC pool):

!!! note
    You can name more than one hostname on the command line, in which case slots for
    **all** of the named hostnames and/or slots are listed.

List all the slots in the pool.

1. How many are there total?
2. How many slots are currently available?

Viewing a Slot ClassAd
----------------------

Just as with `condor_q`, you can use `condor_status` to view the complete ClassAd for a given slot (often confusingly called the “machine” ad):

``` console
[username@ap40]$ condor_status -long <hostname>
```

Because slot ClassAds may have 150–200 attributes (or more), it probably makes the most sense to show the ClassAd for a single slot at a time, as shown above.

Here are some examples of common, interesting attributes taken directly from `condor_status` output:

``` file
OpSys = "LINUX"
DetectedCpus = 96
OpSysAndVer = "CentOS9"
MyType = "Machine"
LoadAvg = 7.0
TotalDisk = 1424814084
TotalMemory = 80000
Machine = "wsu-lm02.osris.org"
CondorVersion = "$CondorVersion: 24.7.3 2025-04-22 BuildID: 803720 PackageID: 24.7.3-1 GitSHA: e207c094 $"
```

As you may be able to tell, there is a mix of attributes about the machine as a whole (hence the name “machine ad”) and about the slot in particular.

Go ahead and examine a machine ClassAd now.

Viewing Slots by ClassAd Expression
-----------------------------------

Often, it is helpful to view slots that meet some particular criteria. For example, if you know that your job needs a lot of memory to run, you may want to see how many high-memory slots there are and whether they are busy. You can filter the list of slots like this using the `-constraint` option and a ClassAd expression.

For example, suppose we want to list all slots that are running Scientific Linux 7 (operating system) and have at least 16 GB memory available. Note that memory is reported in units of Megabytes. The command is:

``` console
[username@ap40]$ condor_status -constraint 'OpSysAndVer == "CentOS7" && Memory >= 16000'
```

!!! note
    Be very careful with using quote characters appropriately in these commands.
    In the example above, the single quotes (`'`) are for the shell, so that the entire expression is passed to
    `condor_status` untouched, and the double quotes (`"`) surround a string value within the expression itself.

Currently on the OSPool, there are only a few slots that meet these criteria.

If you are interested in learning more about writing ClassAd expressions, look at section 4.1 and especially 4.1.4 of the HTCondor Manual. This is advanced material, so it is not required. But if you do, take some time to practice writing expressions for the `condor_status -constraint` command.

!!! note
    The `condor_q` command accepts the `-constraint` option as well!
    As you might expect, the option allows you to limit the jobs that are listed based on a ClassAd expression.

Bonus: Formatting Output
----------------------------

The `condor_status` command accepts the same `-autoformat` (`-af`) options that `condor_q` accepts, and the options have the same meanings in both commands. Of course, the attributes available in machine ads may differ from the ones that are available in job ads. Use the HTCondor Manual or look at individual slot ClassAds to get a better idea of what attributes are available.

For example, you can query the hostname and operating system of the slots with more than 32GB of memory:

``` console
[username@ap40]$ condor_status -af Machine -af OpSysAndVer -constraint 'Memory >= 32000'
```

If you'd like, spend a few minutes now or later experimenting with `condor_status` formatting.

References
----------

As suggested above, if you want to learn more about `condor_status`, you can do some reading:

-   Read the `condor_status` man page or HTCondor Manual section (same text) to learn about more options
-   Read about [ClassAd attributes](https://htcondor.readthedocs.io/en/latest/classad-attributes/index.html) in the appendix of the HTCondor Manual
-   Read about [ClassAd expressions](https://htcondor.readthedocs.io/en/latest/misc-concepts/classad-mechanism.html#old-classads-in-the-htcondor-system) in section 4.1.4 of the HTCondor Manual


