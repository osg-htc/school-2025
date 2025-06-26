# Organizing HTC Workloads

## Exercise Goal

When working with large datasets or running a task many times over (e.g. across subsets, parameters, or files), organizing your workload becomes essential. High-throughput computing (HTC) allows you to run thousands of independent jobs efficiently, but success depends on how well you prepare your files and manage your project structure.

This exercise introduces core principles of HTC workflow organization through a practical example. While the specific task—read mapping—comes from bioinformatics, the underlying strategies apply to any domain that handles data at scale. 

You'll learn how to:

* Plan and organize your workload for high-throughput execution
* Structure input, output, and support files for efficient job management
* Track outputs, logs, and errors across many jobs in a clean, reproducible way


In this exercise, you’ll use a typical bioinformatics read mapping workflow to explore how to sustainably scale your workloads on the OSPool. You’ll focus on job-level organization, building a multi-job submit file, tracking job progress with logs, and troubleshooting job failures.


## Log into an OSPool Access Point

Make sure you are logged into `ap40.uw.osg-htc.org`. 

## Get Files

To get the files for this exercise:

1.  Make a new directory in `~/scaling-up/` and change directory into it
2.  Use pelican to get the input files for our exercises:

        :::console
        osdf object get /ospool/uc-shared/public/school/2025/Celegans_ref.mmi ./
        osdf object get /ospool/uc-shared/public/school/2025/reads.fastq ./
        osdf object get /ospool/uc-shared/public/school/2025/minimap2.sif ./

1.  As you learned earlier, expand this tarball file; it will create a `organizing-files` directory.
1.  Change to that directory, or create a separate one for this exercise and copy the files in.


!!! note "Using Files from the Data Exercises section"

    We will be reusing some of the files from the Data Exercises 2 - OSDF for inputs and OSDF for outputs sections. If you did not complete these or wish to simply get a fresh set of these files, use the command below:
        
        osdf object get /ospool/uc-shared/public/school/2025/Celegans_ref.mmi ./
        osdf object get /ospool/uc-shared/public/school/2025/minimap2.sif ./

## Our Workload

Imagine you're working with a large set of DNA sequencing data from an organism. The sequencing file contains millions of snippets of DNA, which we call **reads**. Your goal is to figure out where these reads came from by matching them to a known reference genome using a read mapping tool.

One commonly used tool for this task is `minimap2`, which quickly finds where each DNA read best matches the reference. For this exercise, we'll use `minimap2` to map our reads to a reference genome.

To do that, we might run a command like:

    :::console
    $ minimap2 -ax map-ont [reference_genome.fasta] [sequencing_reads.fastq] > output.sam

Here’s what these files are:
- `reference_genome.fasta`: a file containing the complete reference genome we’re mapping to.
- `sequencing_reads.fastq`: a file containing millions of DNA fragments, called **reads**, from a sequencing machine.
- `output.sam`: a **SAM file**, which is a standard text format, used to store the results of mapping—where each read aligns in the genome.

We want to run this command to map all our reads against the reference genome. FASTQ files contain 4 lines per read, you can run the following command to calculate the number of reads in your FASTQ file:

    :::console
    $ expr $(wc -l < reads.fastq) / 4
    $ 49382043

Read mapping using algorithms, like `minimap2`, do not scale up well by simply adding additional CPUs to the problem. These mappers typically plateau their speed around 2-4 CPUs. This problem, however, can be solved by employing a "divide and conquer" approach. With this approach, we can subdivide our input reads.fastq file into smaller subsets which can be submitting to HTCondor as a set of independent parallel-running jobs. 

We expect to be working with the following files in each of our jobs:

* **Inputs**
    * A subset of reads.fastq - `reads_subset_a.fastq`
    * The reference genome - `reference_genome.fasta`
    * A minimap2 container image - `minimap2.sif`
    * The executable - `run_minimap2.sh`
* **Outputs**
    * A SAM-formatted output file - `reads_subset_a.sam` 
* **System Generated Files**
    * A set of log, standard error, and standard out files

## Make an Organization Plan

Based on what you know about the script, inputs, and outputs,
how would you organize this HTC workload in directories (folders) on the Access Point?

There will also be system and HTCondor files produced when we submit a job&nbsp;&mdash;
how would you organize the log, standard output, and standard error files?

!!! tip "When to Use the Open Science Data Federation (OSDF)"
    
    Make sure to consider which files will be re-used often (common files across all jobs) versus which files will be used only once. Files often re-used, can be placed in your `/ospool/ap40/data/<user.name>/` directory to take advantage of the caching benefits when using the `osdf://` transfer plugin.

### Organize Files

There are many different ways to organize files;
a simple method that works for most workloads is having a directory for your input files
and a directory for your output files.

For our exercise, we will use the following data organizational structure:

    :::console
    ├── /home/<user.name>/
    │   ├── scaling-up
    │   │   ├── inputs
    │   │   ├── outputs
    │   │   ├── logs
    │   │   │   ├── log
    │   │   │   ├── error
    │   │   │   ├── output
    ├── /ospool/ap40/data/<user.name>/
    │   ├── scaling-up
    │   │   ├── inputs
    │   │   ├── software

1.  Set up this structure on the command line by running: 

        :::console
        $ mkdir -p inputs
        $ mkdir -p outputs
        $ mkdir -p logs
        $ mkdir -p logs/log
        $ mkdir -p logs/error
        $ mkdir -p logs/output
        $ cd /ospool/ap40/data/$USER/
        $ mkdir -p scaling-up/inputs
        $ mkdir -p scaling-up/software

2. Move the `reads.fastq` file to your `inputs` directory (the one under `/home`) using the `mv` command. 

3. Move the `reference_genome.fasta` file to your `/ospool/ap40/data/<user.name>/scaling-up/inputs` directory using the `mv` command. 

4. Move the `minimap2.sif` container image file to your `/ospool/ap40/data/<user.name>/scaling-up/software` directory using the `mv` command.

!!! tip "Stop and Consider: Why are we moving our files to these directories?"

    When preparing your jobs for high-throughput computing, think about how each file will be used. Will the file be reused across many jobs? Or is it specific to a single job? Will it need to be transferred repeatedly—or just once?
    
    With that in mind:  
    - Why might it make sense to place `reads.fastq` in your `/home` directory?  
    - Why are we storing `reference_genome.fasta` and `minimap2.sif` in `/ospool/ap40/data/` instead?

    ??? success "Solution"

        Files in `/home` are transferred to each job, directly from the AP to the EP. These files do not get cached in the OSDF, which makes sense for files that change across jobs—such as subsets of `reads.fastq`. 

        But `reference_genome.fasta` and `minimap2.sif` are the same in every job. By storing them in `/ospool/ap40/data/` and using `osdf://` to access them, we avoid repeatedly transferring them. Instead, they're cached near where jobs run, which speeds things up and reduces network load.


### Wrangling the Data
To get ready for our mapping step, we need to prepare our read files. This includes two crucial steps, splitting our reads.

1. Navigate to your `~/scaling-up/inputs/` directory.
    
        :::console
        cd ~/scaling-up/inputs/

2. Split the FASTQ file into subsets. We can divide our `reads.fastq` into subsets of `500,000` reads per subset. Since each FASTQ read consist of four lines in the FASTQ file, we can split `reads.fastq` every 2,000,000 lines.

        :::console
        split -l 2000000 reads.fastq reads_fastq_chunk_
        rm reads.fastq 

    This will generate 99 read subset files with the prefix `eads_fastq_chunk_`. 

    !!! tip "Subsetting Data - Your Milage May Vary"
    
        When subsetting your data, you should exercise your own judgement. The ideal job profile on the OSPool typically looks something like:
            
        * Runtime: Between 10mins and <10hrs
        * Memory (RAM): 1-5 GB
        * Inputs: <1 GB
        * Outputs: <1 GB
    
        Jobs with larger profiles **may** still run, but will likely face significant increases in `idle` state while waiting for a machine to match. 

3. Delete (or move) the `reads.fastq` file from the input directory.

In the next exercise, we will generate a list of jobs for HTCondor using the the subset files in this directory. To avoid accidentally including the `reads.fastq` file in our list of jobs, please delete it or move it out of this directory.

!!! warning "Organization in HTC is Critical!"
    
    One of the most important steps in scaling up our workflows to run on HTC systems, such as the OSPool, is maintaining a clear organizational structure. This includes deleting files we will not be using anymore. For the rest of the exercise, we will not be using the `reads.fastq` file after splitting it. **Not deleting this file can cause downstream issues. Do not skip this step.**

## Review Your Progress

Now that we've set up our directory structure and pre-processed our data, we can focus on preparing our submission scripts and getting our jobs running!

!!! success "Checking your progress"

    Before you move on, take this time to comb through your directory structure and compare it to the structure below.

    View the current directory and its subdirectories by using the `ls` command with the *recursive* (`-R`) flag. Your overall structure should look something like this:
    
        ├── /home/<user.name>/
        │   ├── scaling-up
        │   │   ├── inputs
        │   │   │   ├── reads_fastq_chunk_a
        │   │   │   ├── reads_fastq_chunk_b
        │   │   │   ├── reads_fastq_chunk_c
        │   │   ├── outputs
        │   │   ├── logs
        │   │   │   ├── log
        │   │   │   ├── error
        │   │   │   ├── output
        ├── /ospool/ap40/data/<user.name>/
        │   ├── scaling-up
        │   │   ├── inputs
        │   │   │   ├── reference_genome.fasta
        │   │   ├── software
        │   │   │   ├── minimap2.sif