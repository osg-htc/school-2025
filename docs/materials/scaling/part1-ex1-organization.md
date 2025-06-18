# Organizing HTC Workloads

Imagine you are a computational biologist working on the mighty roundworm (_C. elegans_). You've conducted a mutagenesis experiment, sequenced your worm genomes, and now want to map your genomic reads to see how different regions of the genome have evolved. 

This section will use a typical read mapping workflow in bioinfomatics to explore how we can sustainably scale up our workloads on the OSPool. These exercises will focus on establishing job-level organization, construction of a multi-job submit file, job progress tracking using log information and troubleshooting. 

## Log into an OSPool Access Point

Make sure you are logged into `ap40.uw.osg-htc.org`. 

## Get Files

To get the files for this exercise:

1.  Type `wget https://github.com/osg-htc/school-2025/raw/main/docs/materials/scaling/files/osgus25-day4-ex11-organizing-files.tar.gz` to download the tarball.
1.  As you learned earlier, expand this tarball file; it will create a `organizing-files` directory.
1.  Change to that directory, or create a separate one for this exercise and copy the files in.

## Our Workload

We can map our genomic reads to the _C. elegans_ reference genome using tools like `minimap2` or `bwa`. For this exercise, we will be using `minimap2`. To map our reads to the reference genome, we would use the command:

    :::console
    $ minimap2 -ax map-ont [reference_genome.fasta] [sequencing_reads.fastq] > output.sam

We want to run this command to map all our reads against the reference genome. FASTq files contain 4 lines per read, you can run the following command to calculate the number of reads in your FASTq file:

    :::console
    $ expr $(wc -l < reads.fastq) / 4
    $ 49382043

Read mapping using algorithms, like `minimap2`, do not scale up well by simply adding additional CPUs to the problem. These mappers typically plateau their speed around 2-4 CPUs. This problem, however, can be solved by employing a "divide and conquer" approach. With this approach, we can subdivide our input reads.fastq file into smaller subsets which can be submitting to HTCondor as a set of independent parallel-running jobs. 

The components of each of our jobs will be:

* **Inputs**
    * A subset of reads.fastq - `reads_subset_a.fastq`
    * A copy of the reference genome - `reference_genome.fasta`
    * A copy of our minimap2 container (provided to you) - `minimap2.sif`
    * A copy of our executable (template provided) - `run_minimap2.sh`
* **Outputs**
    * A SAM-formatted output file - `reads_subset_a.sam`
* **System Generated Files**
    * A set of log, standard error, and standard out files - `job.49302_reads_subset_a.log`, `job.49302_reads_subset_a.err`, `job.49302_reads_subset_a.out`

## Make an Organization Plan

Based on what you know about the script, inputs, and outputs,
how would you organize this HTC workload in directories (folders) on the Access Point?

There will also be system and HTCondor files produced when we submit a job&nbsp;&mdash;
how would you organize the log, standard output, and standard error files?

!!! pro-tip "When to Use the Open Science Data Federation (OSDF)"
    
    Make sure to consider which files will be re-used often (common files across all jobs) versus which files will be used only once. Files often re-used, can be placed in your `/ospool/ap40/data/<user.name>/` directory to take advantage of the caching benefits when using the `osdf://` transfer plugin.

Try making those changes before moving on to the next section of the tutorial.

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
        $ mkdir inputs
        $ mkdir outputs
        $ mkdir logs
        $ mkdir logs/log
        $ mkdir logs/error
        $ mkdir logs/output
        $ mkdir /ospool/ap40/data/<user.name>/scaling-up/inputs
        $ mkdir /ospool/ap40/data/<user.name>/scaling-up/software

2. Move the `reads.fastq` file to your `inputs` directory using the `mv` command. 

3. Move the `reference_genome.fasta` file to your `/ospool/ap40/data/<user.name>/scaling-up/inputs` directory using the `mv` command. 

4. Move the `minimap2.sif` container image file to your `/ospool/ap40/data/<user.name>/scaling-up/software` directory using the `mv` command. 

!!! pro-tip "Frequently Used Files on the OSDF - `reference_genome.fasta` & `minimap2.sif`"
    
    Every one of our jobs will use both the `reference_genome.fasta` and `minimap2.sif` files. These files, due to their frequent usage and larger size, significantly benefit from the OSDF's caching mechanism. In order to use the OSDF file transfer mechanism, we should 1) place them somewhere on our `ospool/ap40/data/<user.name>/` directory and 2) use the `osdf:///` protocol prefix when calling these files in our submit file.

### Wrangling The Data
To get ready for our mapping step, we need to prepare our read files. This includes two crucial steps, splitting our reads and saving the read subset file names to a file.

1. Navigate to your `~/scaling-up/inputs/` directory

   ```
   cd ~/scaling-up/inputs/
   ```

2. Split the FASTQ file into subsets of `5,000` reads per subset. Since each FASTQ read consist of four lines in the FASTQ file, we can split `reads.fastq` every `20,000` lines

    ```
   split -l 20000 reads.fastq reads_fastq_chunk_
   rm reads.fastq
   ```
!!! warning "Organization in HTC is Critical!"
    
    One of the most important steps in scaling up our workflows to run on HTC systems, such as the OSPool, is maintaining a clear organizational structure. This includes deleting files we will not be using anymore. For the rest of the exercise, we will not be using the `reads.fastq` file after splitting it. **Not deleting this file can cause downstream issues. Do not skip this step.**

## Review Our Progress

Now that we've set up our directory structure and pre-processed our data, we can focus on preparing our submission scripts and getting our jobs running! Before we move on, take this time to comb through your directory structure and compare it to the structure below.

!!! success "Checking our progress"

    View the current directory and its subdirectories by using the `ls` command with the *recursive* (`-R`) flag. Your overall structure should look something like this:
    
        ├── /home/<user.name>/
        │   ├── scaling-up
        │   │   ├── inputs
        │   │   │   ├── reads_fastq_chunk_a
        │   │   │   ├── reads_fastq_chunk_b
        │   │   │   ├── reads_fastq_chunk_zzz
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