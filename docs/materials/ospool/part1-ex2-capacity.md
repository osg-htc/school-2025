---
status: testing
---

# OSPool Exercise 1.2: How Much Can I Get?

As noted in the lecture,
contributions to the OSPool typically come from 90–100 sites and therefore vary a great deal.
When you are thinking about a list of computational tasks and how to transform them into OSPool jobs,
you may ask what is possible to get from the OSPool.
In this exercise, you will focus on one resource: memory.
But similar methods could be used for other kinds of resources.

## Getting Information From Free OSPool Slots

Yesterday, you used the `condor_status` command to view information about available slots in the OSPool.
Here is a more precise command that will output the number of CPU cores and memory (in MB)
for each available slot and save that output as a CSV file:

``` console
$ condor_status -avail -const 'Activity == "Idle"' -af:, CPUs Memory > test.csv
```

Let’s examine this data a bit:

1.  Make sure you are logged in to `ap40.uw.osg-htc.org`
1.  Create and change into a new folder for this exercise, for example ospool-ex12
1.  Run the command above
1.  Open the resulting text file using `less`, `nano`, or whatever tool you like

What do you see? At first, it probably appears to be a lot of noisy data!

*   Are you surprised to see so many rows in which the first number —
    the number of CPU cores — is greater than one?
    See below for why.

*   Do you notice repeated rows? For example, there may be many rows of `1, 4096`;
    i.e., one CPU core and 4,096 MB (or 4 GB) or memory.
    Why would that be the case?

### Contributions come in many sizes

As you can see, contributions to the OSPool come in many sizes,
even when just considering the number of CPU cores and memory.
Why is this so and what does it mean for running jobs on the OSPool?

The variation in size mostly comes from the contributing institutions and site owners or admins,
and how they think it is best to allocate capacity to the OSPool.
Some sites prefer to share capacity in very fine-grained units —
one core at a time, with associated memory and disk —
whereas others prefer to share in larger units.
Sites decide on a sharing policy that best fits into how they manage their total capacity,
including how that fits in with local usage.

What does HTCondor do with larger contributions, especially ones with multiple CPU cores?
It is able to break up those larger chunks of capacity into ones that fit researcher jobs that are waiting to run.
Suppose a contribution originally consists of 4 cores and 16 GB of memory.
HTCondor may decide to split off 1 core and 2 GB of memory for a researcher job that needs that much,
leaving 3 cores and 14 GB of memory still available.
When the researcher job is done, HTCondor may reuse the 1 core/2 GB slot for other similar jobs,
or return that capacity back to the original contribution.

Because HTCondor does not give less fractional CPU cores to researcher jobs,
our hypothetical 4 core/16 GB memory contribution can run at most four 1-core jobs.
Memory (and other resources like disk) can be broken up (“partitioned”) more finely.

So, one reason you saw so much variation in available contributions is because contributions are varied!
But also, you asked HTCondor to show _available_ capacity,
which means that you also saw _partial_ contributions
(like the 3 core/14 GB memory example above)
waiting to be further partitioned and given to researcher jobs.

The OSPool is constantly changing,
as contributions come and go,
and as HTCondor matches researcher jobs to available capacity,
breaking up bigger contributions and recombining the smaller pieces and those jobs come and go.

## Analyzing the Data

Now that you better understand the raw data you have collected, analyze it!

This part is mostly up to you. You have a CSV file, and you can import that into a spreadsheet or other software, maybe even a quick script that you write. It may be interesting to plot the data, generate histograms, or calculate other statistics.

Consider limiting yourself to 10–15 minutes for this step. First, here are some tips that may help with data analysis:

*   We tend to talk about memory in GBs these days, but the default unit in HTCondor is MBs.
    You can get the raw data in GBs with a more complex command:

        :::console
        $ condor_status -avail -const 'Activity == "Idle"' -af:, CPUs "eval(Memory / 1024.0)" > cpus-memory-gb.csv

*   For that matter, you can even add a column to the output for memory (in GBs) per core:

        :::console
        $ condor_status -avail -const 'Activity == "Idle"' -af:, CPUs "eval(Memory / 1024.0)" "eval((Memory / 1024.0) / CPUs)" > cpus-memory-percore.csv

*   Finally, if you want to remove duplicate rows,
    you can add a `sort -u` command between `condor_status` and the output file:

        :::console
        $ condor_status [whatever options you want] | sort -u > cpus-memory.csv

Then, here are some specific questions to explore:

*   This exercise is titled “How Much Can I Get?” —
    so, how much memory can you get?
    What are the ranges available, especially if you break it down by number of cores?
    What is “typical”?

*   The “orange table” in the slides
    (and [in our documentation](https://portal.osg-htc.org/documentation/overview/account_setup/is-it-for-you/#computational-fit-on-the-ospool))
    says that ideal jobs use less than a few GBs of memory.
    Clearly, more than that is available.
    Why do ideal jobs stay within a few GB?

*   Related to the previous questions,
    if you calculate the available memory _per core_ (i.e., memory / cores),
    what are the ranges available and what is typical?
    Why might this be a more relevant analysis?

*   Thought experiment:
    If a contribution consists of, say, 24 cores and 156 GB memory,
    what happens if one single-core job takes all 156 GB memory?
    What is left of the contribution?
    What do you think HTCondor does with the remaining capacity?

Discuss your analysis with neighbors and staff!

## What About Disk Capacity?

You could certainly run the same kinds of commands and do the same kinds of analysis
for disk capacity (in KB) instead of memory.
However, the amount of variation in disk is orders of magnitude greater than for memory,
and so this is probably not very interesting.

Also to some extent, it is less interesting to ask about disk capacity.
In short, disk is cheap and there tends to be a lot of it.
And if you look carefully at the description of “Ideal Jobs” in the table,
you will see we say nothing about disk needs,
but rather focus on the amount of data to be transferred in and out of the Execution Point.
That is because network capacity is usually a greater bottleneck than disk!
There will be more on this topic on Wednesday.
