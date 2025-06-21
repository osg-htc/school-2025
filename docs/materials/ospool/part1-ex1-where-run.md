---
status: testing
---

# OSPool Exercise 1.1: Where Do Jobs Run?

In the lecture about OSG services,
we said that the OSPool is really just a big, strange HTCondor pool.
Yesterday, you already learned a great deal about HTCondor,
and even ran your jobs in the OSPool.
So in a certain way, you will not be doing anything new in these exercises.

Instead, let’s do a bit of research _on the OSPool itself_,
to learn more about this strange environment.
Doing these exercises will help you better understand how to get the most out of the OSPool
for your own research or the research you support or teach.

## Locating Your Jobs

During the lecture, you saw the (partial) map of the institutions that contribute to the OSPool.
But where do your jobs run in practice?
Does each one go to a different location?
Do they clump together?
Do they all run at the nearest institution?
Let’s do an experiment!

There is a single computational task here, and you will run it 50 times to collect a sample of results.
The task simply returns information about the geographic location of the Execution Point it runs on.
We provide the executable and associated data,
so work will be to write a submit file that queues multiple jobs.
Once complete, you will manually combine the results and input them to a mapping service.

### Where in the world are my jobs?

To find the physical location of the computers your jobs our running on, you will use a method called *geolocation*.
Geolocation uses a registry to match a computer’s network address to an approximate latitude and longitude.

To start, you will reuse some basic HTCondor skills from the HTC exercises:

1.  Log in to `ap40.uw.osg-htc.org` (if not already)
1.  Create and change into a new folder for this exercise, for example `osg-ex11`
1.  Download the geolocation code:

        :::console
        $ osdf object get /ospool/uc-shared/public/school/2025/dev/location-wrapper.sh ./
        $ osdf object get /ospool/uc-shared/public/school/2025/dev/wn-geoip.tar.gz ./

    You will be using `location-wrapper.sh` as your executable and `wn-geoip.tar.gz` as an input file.

1.  Create a submit file that queues 50 jobs to run `location-wrapper.sh`,
    transfer `wn-geoip.tar.gz` as an input file,
    and use the `$(Process)` macro to write different `output` and `error` files.
    Also, add the following requirement to the submit file (it’s not important to know what it does, yet):

        container_image = osdf:///ospool/uc-shared/public/school/2025/dev/python27.sif

    Try to do this step without looking at materials from the earlier exercises.
    But if you are stuck, see [HTC Exercise 2.2](../htcondor/part2-ex2-queue-n.md).

1.  Submit your jobs and wait for the results

### Combining your results

When your jobs finish, it’s time to combine them.
Rather than inspecting each output file individually,
you can use the `cat` command to print the results from all of your output files at once.
If all of your output files have the format `location-#.out` (e.g., `location-10.out`),
your command will look something like this:

``` console
$ cat location-*.out
```

The * is a wildcard so the above cat command runs on all files that start with location- and end in .out. Additionally, you can use cat in combination with the sort and uniq commands using "pipes" (|) to print only the unique results:

The `*` is a wildcard so the above cat command runs on all files that start with `location-` and end in `.out`.
Additionally, you can use `cat` in combination with the `sort` and `uniq` commands using “pipes” (`|`)
to print only the unique results:

``` console
$ cat location-*.out | sort | uniq
```

## Mapping your results

To see the locations of the Execution Points that your jobs ran on,
use the https://www.mapcustomizer.com/ website.
Copy and paste the combined (unique) results into the text box that pops up
when clicking on the “Bulk Entry” button on the right-hand side.

Where did your jobs run?
Did they spread out a lot, clump together, all go to the nearest institution, or something else?
At this point in your learning, do you have any ideas about why they ran where they did?
If you have time, talk to your neighbor(s) about it!

## Next exercise

Once completed, move onto the next exercise: [How Much Can I Get?](part1-ex2-how-much-capacity.md).

## Extra Challenge: Cleaning up your submit directory

If you run `ls` in the directory from which you submitted your job, you may see that you now have a hundred files or so.
Proper data management starts to become a requirement as you start to scale up;
it may be helpful to separate your submit files, code, and input data from your output data.

1.  Try editing your submit file so that all your output and error files are saved to separate directories within your
    submit directory.

    !!! note "Tip"
        Experiment with just one or two jobs per submission until you’re confident you have it right,
        then go back to submitting 50 jobs.
        Remember: Test small and scale up!

1.  Submit your file and track the status of your jobs.

Did your jobs complete successfully with output and error files saved in separate directories?
If not, can you find any useful information in the job logs or hold messages?
If you get stuck, review the [slides from Monday](../index.md).
