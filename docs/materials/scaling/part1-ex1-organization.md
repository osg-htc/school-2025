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

>[!TIP]
> Make sure to consider which files will be re-used often (common files across all jobs) versus which files will be used only once. Files often re-used, can be placed in your `/ospool/ap40/data/<user.name>/` directory to take advantage of the caching benefits when using the `osdf://` transfer plugin.

Try making those changes before moving on to the next section of the tutorial.

## Organize Files

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

2. Move the `reads.fastq` file to your `inputs` directory using the `mv` command. 

3. Move the `reference_genome.fasta` file to your `/ospool/ap40/data/<user.name>/scaling-up/inputs` directory using the `mv` command. 

4. . Move the `minimap2.sif` container image file to your `/ospool/ap40/data/<user.name>/scaling-up/software` directory using the `mv` command. 

>[!TIP]
> Every one of our jobs will use both the `reference_genome.fasta` and `minimap2.sif` files. These files, due to their frequent usage and larger size, significantly benefit from the OSDF's caching mechanism. 



3View the current directory and its subdirectories by using the `ls` command with the *recursive* (`-R`) flag:

        :::console
        $ ls -R
        README.md    books.submit input        output       wordcount.py

        ./input:
        Alice_in_Wonderland.txt Huckleberry_Finn.txt
        Dracula.txt             Pride_and_Prejudice.txt

        ./output:

3.  Next, create directories for the HTCondor log, standard output, and standard output files (in one directory):

        :::console
        $ mkdir logs
        $ mkdir errout

## Submit One Job

Now we want to submit a test job that uses this organizing scheme,
using just one item in our input set&nbsp;&mdash;
in this example, we will use the `Alice_in_Wonderland.txt` file from our `input` directory.

1.  Fill in the incomplete lines of the submit file, as shown below:

        :::console
        executable    = wordcount.py
        arguments     = Alice_in_Wonderland.txt

        transfer_input_files    = input/Alice_in_Wonderland.txt
        transfer_output_files   = counts.Alice_in_Wonderland.txt
        transfer_output_remaps  = "counts.Alice_in_Wonderland.txt=output/counts.Alice_in_Wonderland.txt"

    To tell HTCondor the location of the input file, we need to include the input directory.
    Also, this submit file uses the `transfer_output_remaps` feature that you learned about;
    it will move the output file to the `output` directory by renaming or remapping it.

1.  Next, edit the submit file lines that tell the log, output, and error files where to go:

        :::console
        output        = logs/job.$(ClusterID).$(ProcID).out
        error         = errout/job.$(ClusterID).$(ProcID).err
        log           = errout/job.$(ClusterID).$(ProcID).log

1.  Submit your job and monitor its progress.

## Submit Multiple Jobs

Now, you are ready to submit the whole workload.

1.  Create a file with the list of input files (the input set);
    here, this is the list of the book files to analyze.
    Do this by using the shell `ls` command and redirecting its output to a file:

        :::console
        $ ls input > booklist.txt
        $ cat booklist.txt

1.  Modify the submit file to reference the file of inputs and replace the fixed value (`Alice_in_Wonderland.txt`) with a variable (`$(book)`):

        :::console
        executable    = wordcount.py
        arguments     = $(book)

        transfer_input_files    = input/$(book)
        transfer_output_files   = counts.$(book)
        transfer_output_remaps  = "counts.$(book)=output/counts.$(book)"

        queue book from booklist.txt

1.  Submit the jobs

1.  When complete, look at the complete set of input and (now) output files to see how they are organized.
