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

!!! tip
    
    Make sure to consider which files will be re-used often (common files across all jobs) versus which files will be used only once. Files often re-used, can be placed in your `/ospool/ap40/data/<user.name>/` directory to take advantage of the caching benefits when using the `osdf://` transfer plugin.

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
        $ mkdir /ospool/ap40/data/<user.name>/scaling-up/software

2. Move the `reads.fastq` file to your `inputs` directory using the `mv` command. 

3. Move the `reference_genome.fasta` file to your `/ospool/ap40/data/<user.name>/scaling-up/inputs` directory using the `mv` command. 

4. Move the `minimap2.sif` container image file to your `/ospool/ap40/data/<user.name>/scaling-up/software` directory using the `mv` command. 

!!! tip
    
    Every one of our jobs will use both the `reference_genome.fasta` and `minimap2.sif` files. These files, due to their frequent usage and larger size, significantly benefit from the OSDF's caching mechanism. In order to use the OSDF file transfer mechanism, we should 1) place them somewhere on our `ospool/ap40/data/<user.name>/` directory and 2) use the `osdf:///` protocol prefix when calling these files in our submit file.

5. View the current directory and its subdirectories by using the `ls` command with the *recursive* (`-R`) flag:

        :::console
        $ ls -R
        README.md    books.submit input        output       wordcount.py

        ./input:
        Alice_in_Wonderland.txt Huckleberry_Finn.txt
        Dracula.txt             Pride_and_Prejudice.txt

        ./output:

## Composing Your Job

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
!!! warning "Important"
    
    One of the most important steps in scaling up our workflows to run on HTC systems, such as the OSPool, is maintaining a clear organizational structure. This includes deleting files we will not be using anymore. For the rest of the exercise, we will not be using the `reads.fastq` file after splitting it. **Not deleting this file can cause downstream issues. Do not skip this step.**

### Adapting the Executable
Now that we have our data partitioned into independent subsets to be mapped in parallel, we can work on adapting our executable for use on the OSPool. We will start with the following template executable file, which is also found in your project directory under `~/scaling-up/minimap2.sh`.

    :::console
    #!/bin/bash
    # Use minimap2 to map the basecalled reads to the reference genome
    ./minimap2 -ax map-ont reference_genome.fasta reads.fastq > output.sam

| Command Segment | ./minimap2                             | -ax map-ont                                                                      | reference_genome.fasta               | reads.fastq                                 | \>                                         | output.sam                          |
|-----------------|----------------------------------------|----------------------------------------------------------------------------------|--------------------------------------|---------------------------------------------|--------------------------------------------|-------------------------------------|
| **Meaning**         | The program we'll run to map our reads | Specifies the type of reads we're using <br>(Oxford Nanopore Technologies reads) | The input reference we're mapping to | The reads we are mapping against our genome | redirects the output of minimap2 to a file | The output file of our mapping step |

!!! question "Time-Out!"

    **Before moving forward think about how you would adapt this executable template for HTC**
    If we want to map each one of our reads subsets against the reference genome, think about the following questions:
        * What parts of the command will change with each job?
        * What parts of the command will stay the same?

Let's start by editing our template executable file! In our executable there's two main segments of the `minimap2` command that will be changed: The input `reads.fastq` file and the output `output.sam` file. 

>[!IMPORTANT]
!!! question "Thinking Ahead Before Errors!"

    **Renaming our output files**
    What do you think would happen if we do keep the output file on our executable as `output.sam`?

1. Modify the executable to accept the name of our input `reads.fastq` subsets as an argument.

        :::console
        #!/bin/bash
        reads_subset_file = $1
        # Use minimap2 to map the basecalled reads to the reference genome
         ./minimap2 -ax map-ont reference_genome.fasta $(reads_subset_file) > output.sam

2. Modify the executable use the name of our input reads subset file (`$reads_subset_file`) as the prefix of our output file.

        :::console
        #!/bin/bash
        reads_subset_file = "$1"
        # Use minimap2 to map the basecalled reads to the reference genome
         ./minimap2 -ax map-ont reference_genome.fasta "$(reads_subset_file)" > "$(reads_subset_file)_output.sam"

### Generating the List of Jobs
Next, we need to generate a list of jobs for HTCondor to run. In previous exercises, we've used the `queue` statements such as `queue <num>` and `queue <variable> matching *.txt`. For our exercise, we will use the `queue <var> from <list>` submission strategy. 

??? question 
    What values should we pass to HTCondor to scale our `minimap2` workflow up? 

1. Move to your `~/scaling-up/inputs/` directory

        :::console
        $ mv ~/scaling-up/inputs/
        $ ls -la
        total 12
        drwxr-xr-x  2 daniel.morales.1 daniel.morales.1 4096 Jun 13 16:08 .
        drwx------ 10 daniel.morales.1 daniel.morales.1 4096 Jun 13 16:07 ..
        -rw-r--r--  1 daniel.morales.1 daniel.morales.1   14 Jun 13 16:08 reads_fastq_chunk_a
        -rw-r--r--  1 daniel.morales.1 daniel.morales.1   14 Jun 13 16:08 reads_fastq_chunk_b
        -rw-r--r--  1 daniel.morales.1 daniel.morales.1   14 Jun 13 16:08 reads_fastq_chunk_c
        -rw-r--r--  1 daniel.morales.1 daniel.morales.1   14 Jun 13 16:08 reads_fastq_chunk_d
2. Make a list of all the files in `~/scaling-up/inputs/` and save it to `~/scaling-up/list_of_fastq.txt`

        :::console
        $ ls > ~/scaling-up/list_of_fastq.txt
        $ mv ~/scaling-up/
        $ ls -la
        total 12
        drwxr-xr-x  2 daniel.morales.1 daniel.morales.1 4096 Jun 13 16:08 .
        drwx------ 10 daniel.morales.1 daniel.morales.1 4096 Jun 13 16:07 ..
        -rw-r--r--  1 daniel.morales.1 daniel.morales.1   14 Jun 13 16:08 list_of_fastq.txt

3. Use `head` to preview the first 10 lines of `list_of_fastq.txt`

        :::console
        $ head ~/scaling-up/list_of_fastq.txt
        reads_fastq_chunk_a
        reads_fastq_chunk_b
        reads_fastq_chunk_c
        reads_fastq_chunk_d
        reads_fastq_chunk_e
        ...
        reads_fastq_chunk_j

## Testing Our Jobs - Submit **_One_** Job

Now we want to submit a test job that uses this organizing scheme,
using just one item in our input set&nbsp;&mdash;
in this example, we will use the `Alice_in_Wonderland.txt` file from our `input` directory.

!!! tip "Time-Out!"

    It is important to always check your workflows before scaling up to full production. Generally, we recommand testing your job with a single job followed by a handful (2-5) jobs before submitting your full submission. 

1.  Fill in the incomplete lines of the submit file, as shown below:

        :::console
        +SingularityImage       = "osdf:///ospool/ap40/data/<user.name>/scaling-up/software/minimap2.sif"

        executable              = minimap2.sh
        arguments               = reads_fastq_chunk_a

        transfer_input_files    = ./input/reads_fastq_chunk_a, osdf:///ospool/ap40/data/<user.name>/scaling-up/inputs/reference_genome.fasta
        transfer_output_files   = reads_fastq_chunk_a_output.sam
        transfer_output_remaps  = "reads_fastq_chunk_a_output.sam=output/reads_fastq_chunk_a_output.sam"

    To tell HTCondor the location of the input file, we need to include the input directory.
    Also, this submit file uses the `transfer_output_remaps` feature that you learned about;
    it will move the output file to the `output` directory by renaming or remapping it.

2.  Next, edit the submit file lines that tell the log, output, and error files where to go:

           :::console
           output        = logs/output/job.$(ClusterID).$(ProcID)_reads_fastq_chunk_a_output.out
           error         = logs/error/job.$(ClusterID).$(ProcID)_reads_fastq_chunk_a_output.err
           log           = logs/log/job.$(ClusterID).$(ProcID)_reads_fastq_chunk_a_output.log

3.  Last, add to the submit file your resource requirements:

        :::console
        request_cpus           = 2
        request_disk           = 4 GB
        request_memory         = 4 GB 
     
        queue 1

4.  Submit your job and monitor its progress.

## Submit Multiple Jobs

Now, you are ready to submit the whole workload. 

!!! example "Try It Yourself!"

    You've split your large FASTQ file into multiple read subsets, and you're ready to run `minimap2` on all of them in parallel.

    1. Edit your submit file to use the `queue <var> from <file>` syntax.
    2. Ensure the `arguments`, `transfer_input_files`, `transfer_output_files` fields change with each input.
    3. Ensure your `log`, `error`, and `output` files all include the name of the read subset file being used mapped in this job.
    4. Organize the output files using the correct `transfer_output_remaps` statement
    
    🧠 Before submitting:
    - Are all your subset filenames listed in `list_of_fastq.txt`?
      - Did you test at least one job successfully?
      - Are you remapping outputs into the `outputs/` folder?
    
    When ready, submit with:
    ```bash
    condor_submit minimap2_multi.submit
    ```
    
    ??? example "Solution Set"

        Your final submit file should look something like this:
        
        :::console
        +SingularityImage      = "osdf:///ospool/ap40/data/<user.name>/scaling-up/software/minimap2.sif"

        executable             = ./minimap2.sh
        arguments              = $(read_subset_file)
        transfer_input_files   = ./input/$(read_subset_file), osdf:///ospool/ap40/data/<user.name>/scaling-up/inputs/reference_genome.fasta
        
        transfer_output_files  = ./$(read_subset_file)_output.sam
        transfer_output_remaps  = "$(read_subset_file)_output.sam=output/$(read_subset_file)_output.sam"
         
        output        = logs/output/job.$(ClusterID).$(ProcID)_$(read_subset_file)_output.out
        error         = logs/error/job.$(ClusterID).$(ProcID)_$(read_subset_file)_output.err
        log           = logs/log/job.$(ClusterID).$(ProcID)_$(read_subset_file)_output.log

        request_cpus           = 2
        request_disk           = 4 GB
        request_memory         = 4 GB 
         
        queue read_subset_file from ./list_of_fastq.txt