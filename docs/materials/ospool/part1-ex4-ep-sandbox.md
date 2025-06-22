---
status: testing
---

# OSPool Exercise 1.4: What Is In an Execution Point?

When using the OSPool, or any HTCondor pool,
it can be difficult to build a mental model of the Execution Point and
what is going on there while your job is running.
In particular, it may be hard to imagine what your working directory on the Execution Point is like and
how that compares to your working directory on the Access Point.
So let’s explore both!

## Capturing Details of Execution Point Directories

In this exercise, you will run jobs on the OSPool
to collect information about what Execution Point “sandbox” directories look like.
Every time that HTCondor starts your job on an Execution Point, it creates and changes to a “sandbox” directory —
your job’s temporary working space within the Execution Point.

Here is a simple shell script that captures a bit of data about the sandbox directory it is running in:

``` file
#!/bin/sh

# Print data elements to standard out:
# - Current date and time of the system running this script
# - Fully-qualified (long) hostname of the system
# - Path to the current working directory
# - The site name of the system
echo "$(date),$(hostname -f),$(pwd),${OSG_SITE_NAME}"

# Dump a recursive directory listing of the current working directory to an output file
ls -lR "`pwd`" > output.txt 2>&1
```

Here’s what to do:

1.  Make sure you are logged in to `ap40.uw.osg-htc.org`
1.  Create and change into a new folder for this exercise, for example ospool-ex14
1.  Create a file named `fs-probe.sh` with the shell script above
1.  Create an HTCondor submit file to run this script

    Tips: here are no command-line arguments and no input file.
    Save standard output and error to filenames that include the HTCondor process ID.
    Request 1 GB of memory and just 200 KB of disk.
    If you are not sure about how to do these things, look at Monday’s slides or exercises, or ask around.

1.  Add special commands to the submit file to handle the output file(s)

    Because the script creates an output file with the same name in each job,
    not only do we need to transfer the file back but we need to rename it, too.
    You’ll learn more about this technique tomorrow,
    but for now, just add these two lines to your submit file:

        :::console
        transfer_output_files = output.txt
        transfer_output_remaps = "output.txt = fsprobe-$(PROCESS)-ls-output.txt"

1.  Try running just one job like this and fix any issues
1.  When ready, run 20–30 copies of this job in one submission

## Reviewing the Data

The script produces CSV output in the standard output files and
a recursive directory listing of the Execution Point sandbox directory in each `fsprobe-*-ls-output.txt`.
Let’s examine each type of output in turn.

### CSV output

You can combine the CSV outputs as follows (assuming you named your output files `something-ProcID.out`):

```
cat *.out > combined-output.csv
```

Look at that combined file directly or, probably better yet,
copy the contents to a CSV file on your laptop and open it as a spreadsheet.
Use the script comments to understand what each column is, but in summary:
timestamp, EP hostname, sandbox directory path, site name.

The key here is to look through the sandbox directory paths.
Are they all the same?
Are any of them the same as your working directory for this exercise on the Access Point?
Why or why not?

!!! note
    Do you see some sandbox paths that are just `/srv`?
    If so, that job ran inside a default OSPool container (that we provided).
    You will learn more about containers later today,
    but the key idea here is that your job cannot see the host filesystem outside of the container,
    and we (as creators of the container image) arbitrarily picked `/srv` as the sandbox directory.

### Directory Listings

Look through each of the directory listing files (`fsprobe-N-ls-output.txt`).
(Fun Linux hack: You can view them all at once by running `less *ls-output.txt` and then
typing `:n` to go to the next file in the set or `:p` to go backward;
the current filename is at the lower left and `q` quits the less program.)

Each directory listing captures what files and directories (recursively) were in the sandbox directory
at the moment the shell script ran the `ls` command.
What was there? What _was not_ there?
How are these directories similar to and different from each other.
How are they similar to and different from your working directory on the Access Point.
Think about these things when preparing and running jobs for the OSPool!
