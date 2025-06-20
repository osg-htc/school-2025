---
status: reviewed
---

Data Exercise 2.1: Using OSDF for Large Shared Data
===================================================

This exercise will use a [Minimap2](https://github.com/lh3/minimap2) workflow to
demonstrate the functionality of OSDF for transferring input files to jobs on OSG.

As presented in the Data [lecture](files/osgus25-data.pdf), for submitting multiple jobs that require larger files, OSDF not only reduces the load on the OSPool network
but also keeps a copy of the files in different caching sites, which results in faster transfer of files to the Execution Point. 

OSDF is connected to a distributed set of caches spread across the U.S.
They are connected with high bandwidth connections to each other, and to the data origin servers, where your data is
originally placed.

![OSDF Map](files/osgus19-day4-part2-CacheLocations.png)

Setup
-----
- Please make sure that you are logged into `ap40.uw.osg-htc.org`.
- Create a folder named `minimap2-read` and `cd` into the directory
- All the required files for this lesson are located at `/ospool/ap40/osg-staff/tutorial-ospool-minimap/`
- Copy all the files into the `minimap2-read` directory. 

!!! note "File Sizes"
    - What are the contents and the size for the `data` and `software` directories that you copied?

### Software Setup
For using Minimap2, you will need a container of minimap2. The container is provided in the `/ospool/ap40/osg-staff/tutorial-ospool-minimap/software` directory. 

Place the Database in OSDF
--------------------------------

### Copy to your data to the OSDF space

OSDF provides a directory for you to store data which can be accessed through the caching servers.
First, you need to move your `minimap2.sif` image. For `ap40.uw.osg-htc.org`, the directory
to use is `/ospool/ap40/data/[USERNAME]/`

Note that files placed in the `/ospool/ap40/data/[USERNAME]/` directory will only be accessible
by your own jobs.

Modify the Submit File and Wrapper
----------------------------------

You will have to modify the wrapper and submit file to use OSDF:

1. HTCondor knows how to do OSDF transfers, so you just have to provide the correct URL in 
   `transfer_input_files`. Note there is no servername (3 slashes in :///) and we instead
   is is just based on namespace (`/ospool/ap40` in this case):

        ::file
        transfer_input_files = blastx, $(inputfile), osdf:///ospool/ap40/data/[USERNAME]/pdbaa_files.tar.gz

1. Confirm that your queue statement is correct for the current directory. It should be something like:

        ::file
        queue inputfile matching mouse_rna.fa.*

And that `mouse_rna.fa.*` files exist in the current directory (you should have copied a few them from the previous exercise
directory).

Submit the Job
--------------

Now submit and monitor the job! If your 100 jobs from the previous exercise haven't started running yet, this job will
not yet start.
However, after it has been running for ~2 minutes, you're safe to continue to the next exercise!

Considerations
--------------

1. Why did we not place all files in OSDF (for example, `blastx` and `mouse_rna.fa.*`)?

1. What do you think will happen if you make changes to `pdbaa_files.tar.gz`? Will the caches
   be updated automatically, or is there a possiblility that the old version of
   `pdbaa_files.tar.gz` will be served up to jobs? What is the solution to this problem?
   (Hint: OSDF only considers the filename when caching data)

Note: Keeping OSDF 'Clean'
--------------------------------

Just as for any data directory, it is VERY important to remove old files from OSDF when you no longer need them,
especially so that you'll have plenty of space for such files in the future.
For example, you would delete (`rm`) files from `/ospool/ap40/data/[USERNAME]/` on when you don't need them there
anymore, but only after all jobs have finished.
The next time you use OSDF after the school, remember to first check for old files that you can delete.

Next exercise
-------------

Once completed, move onto the next exercise: [Using OSDF for outputs](part2-ex2-osdf-outputs.md)

