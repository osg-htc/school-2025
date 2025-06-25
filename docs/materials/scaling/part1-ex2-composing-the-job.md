# Composing Your Jobs

## Exercise Goal

In our previous exercise, [Scaling-Up Exercise 1 Part 1](../part1-ex1-organization), we learned about the importance of preparing and organizing a directory structure for large-scale workloads. In this section, we'll learn strategies to compose and test these large-scale workloads in the form of jobs.

## Introduction

High throughput computing allows us to efficiently scale analyses by distributing jobs across many computing resources. In this lesson, we will continue the example from the previous exercise, now learning how to structure and submit a read mapping workflow using the OSPool and `minimap2`. This includes adapting your executable script and submit file to dynamically handle many input files in parallel.

!!! danger "**Halt!** Do not proceed if you haven't completed the [Scaling-Up Exercise 1 Part 1](../part1-ex1-organization)"
    
    This is part two of our Scaling Up Exercise 1 set and should **only** be completed after you've successfully completed [Scaling-Up Exercise 1 Part 1](../part1-ex1-organization). 

## Log into an OSPool Access Point

Make sure you are logged into `ap40.uw.osg-htc.org`. 

## Generating the List of Jobs
Next, we need to generate a list of jobs for HTCondor to run. In previous exercises, we've used the `queue` statements such as `queue <num>` and `queue <variable> matching *.txt`. 

For our exercise, we will use the `queue <var> from <list>` submission strategy. 

!!! tip "Think Ahead!" 
    What values should we pass to HTCondor to scale our `minimap2` workflow up? 

1. Move to your `~/scaling-up/inputs/` directory
        
        $ mv ~/scaling-up/inputs/
        $ ls -la
        total 12
        drwxr-xr-x  2 username username 4096 Jun 13 16:08 .
        drwx------ 10 username username 4096 Jun 13 16:07 ..
        -rw-r--r--  1 username username   14 Jun 13 16:08 reads_fastq_chunk_a
        -rw-r--r--  1 username username   14 Jun 13 16:08 reads_fastq_chunk_b
        -rw-r--r--  1 username username   14 Jun 13 16:08 reads_fastq_chunk_c
        -rw-r--r--  1 username username   14 Jun 13 16:08 reads_fastq_chunk_d

2. Make a list of all the files in `~/scaling-up/inputs/` and save it to `~/scaling-up/list_of_fastq.txt`

        :::console
        $ ls > ~/scaling-up/list_of_fastq.txt
        $ mv ~/scaling-up/
        $ ls -la
        total 12
        drwxr-xr-x  2 username username 4096 Jun 13 16:08 .
        drwx------ 10 username username 4096 Jun 13 16:07 ..
        -rw-r--r--  1 username username   14 Jun 13 16:08 list_of_fastq.txt

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

## Adapting the Executable
Now that we have our data partitioned into independent subsets to be mapped in parallel, we can work on adapting our executable for use on the OSPool. We will start with the following template executable file, which is also found in your project directory under `~/scaling-up/minimap2.sh`.

    :::console
    #!/bin/bash
    # Use minimap2 to map the basecalled reads to the reference genome
    minimap2 -ax map-ont reference_genome.fasta reads.fastq > output.sam

| Command Segment | `minimap2`                             | `-ax map-ont`                                                                      | `reference_genome.fasta`               | `reads.fastq`                                 | `>`                                         | `output.sam`                          |
|-----------------|----------------------------------------|----------------------------------------------------------------------------------|--------------------------------------|---------------------------------------------|--------------------------------------------|-------------------------------------|
| **Meaning**         | The program we'll run to map our reads | Specifies the type of reads we're using <br>(Oxford Nanopore Technologies reads) | The input reference we're mapping to | The reads we are mapping against our genome | redirects the output of minimap2 to a file | The output file of our mapping step |

!!! danger "Time-Out! **Think about how you would adapt this executable template for HTC**"

    If we want to map each one of our reads subsets against the reference genome, think about the following questions:

    * What parts of the command will change with each job?
    * What parts of the command will stay the same?

Let's start by editing our template executable file! In our executable there's two main segments of the `minimap2` command that will be changed: The input `reads.fastq` file and the output `output.sam` file. 

!!! warning "Thinking Ahead Before Errors! **Renaming our output files**"

    What do you think would happen if we do keep the output file on our executable as `output.sam`?

1. Modify the executable to accept the name of our input `reads.fastq` subsets as an argument.

        :::console
        #!/bin/bash
        reads_subset_file="$1"
        # Use minimap2 to map the basecalled reads to the reference genome
        minimap2 -ax map-ont reference_genome.fasta $(reads_subset_file) > output.sam

2. Modify the executable use the name of our input reads subset file (`$reads_subset_file`) as the prefix of our output file.

        :::console
        #!/bin/bash
        reads_subset_file="$1"
        # Use minimap2 to map the basecalled reads to the reference genome
        minimap2 -ax map-ont reference_genome.fasta "$(reads_subset_file)" > "$(reads_subset_file)_output.sam"

!!! question "Not sure how variables work on bash?"

    Reach out for help from one of the School staff members! You can also review the Software Carpentries' [Unix Shell - Loops](https://swcarpentry.github.io/shell-novice/05-loop.html) tutorial for examples on how to use these variables in your daily computational use.

## Testing Our Jobs - Submit a Test List of Jobs

Now we want to submit a test job with our organizing scheme and adapted executable, using only a small set of our reads subset. We're going to start off with the multi-job submit template below. 

    :::console
    container_image         = <path_to_sif>
    
    executable              = <path_to_executable>

    transfer_input_files    = <path_to_input_files>
    transfer_output_files   = <path_to_output_files>

    log                     = <path_to_log_file>
    error                   = <path_to_stderror_file>
    output                  = <path_to_stdout_file>

    request_cpus            = <num-of-cpus>
    request_memory          = <amount-of-memory>
    request_disk            = <amount-of-disk>

    queue

!!! example "Try It Yourself!"

    You've split your large FASTQ file into multiple read subsets, and you're ready to run `minimap2` on all of them in parallel. Before moving forward, check your understanding by trying to write the submit file yourself! Consider the following:

    1. What `queue` strategy discussed in the OSG School is best for our setup?
        * Think about the List Of Jobs created in the [Generating the List of Jobs](#generating-the-list-of-jobs) section
    2. How can we dynamically specify the `arguments`, `transfer_input_files`, `transfer_output_files` fields values with each `read_subset_file`.
    3. Ensure your `log`, `error`, and `output` files all include the name of the read subset file being used mapped in this job.
    4. Organize the output files using the correct `transfer_output_remaps` statement
        * Remember, we want our output to be saved as `~/scaling-up/outputs/reads_fastq_chunk_a_output.sam` on the Access Point
    5. Which file transfer protocols should we use for our inputs/outputs?
        * Consider whether these files are used one or repeatedly across all your jobs.

    !!! danger "Try to Draft a Submit File Before Moving Forward️"

For our template, lets use `read_subset_file` as our variable name to pass the name of each subset file to.

1. It is useful to think about constructing your submit file starting with the `queue` statement. This helps us predict how we need to structure our submit file in order to dynamically submit many jobs at once. 

    !!! question "What `queue` statement do you think would work best for our current workflow?"
        
        Consider the following:

        * We have a set of FASTQ formatted reads_fastq_chunk_ subset files we need to submit along with each job.
        * Each reads_fastq_chunk_ subset file name is in our `~/scaling-up/list_of_fastq.txt` list of jobs.

        ??? success "Solution"

            Since we have our list of job files saved in `~/scaling-up/list_of_fastq.txt`, we will use the `queue <var> from <list_of_var.txt>` syntax. 

            **Our queue statement will be:**
            
            `queue read_subset_file from ~/scaling-up/list_of_fastq.txt`

2. Fill in the incomplete lines of the submit file, as shown below: 

        :::console
        container_image         = "osdf:///ospool/ap40/data/<user.name>/scaling-up/software/minimap2.sif"

        executable              = minimap2.sh
        arguments               = $(read_subset_file)

3. Now, specify where the input and output files should be:

        :::console
        transfer_input_files    = ./input/$(read_subset_file), osdf:///ospool/ap40/data/<user.name>/scaling-up/inputs/reference_genome.fasta
        transfer_output_files   = $(read_subset_file)_output.sam
        transfer_output_remaps  = "$(read_subset_file)_output.sam=output/$(read_subset_file)_output.sam"

    To tell HTCondor the location of the input file, we need to include the input directory.
    Also, this submit file uses the `transfer_output_remaps` feature that you learned about;
    it will move the output file to the `output` directory by renaming or remapping it.

4.  Next, edit the submit file lines that tell the log, output, and error files where to go:

        :::console 
        output        = logs/output/job.$(ClusterID).$(ProcID)_$(read_subset_file)_output.out
        error         = logs/error/job.$(ClusterID).$(ProcID)_$(read_subset_file)_output.err
        log           = logs/log/job.$(ClusterID).$(ProcID)_$(read_subset_file)_output.log

5.  Add to the submit file your resource requirements:

        :::console
        request_cpus           = 2
        request_disk           = 4 GB
        request_memory         = 4 GB 
     
6.  Lastly, finish your submit file with the `queue` statement:

        queue read_subset_file from ./test_list_of_fastq.txt
    
    We will be using `./test_list_of_fastq.txt` instead while will only have a sample (3) of our reads subsets. We will use the fully scale list in the next section. 

    !!! tip "Thinking of our jobs as a `for` or `while` loop"
        
        We can think of our multi-job submission as a sort of `for` or `while` loop in bash.
        
        !!! success ""
            **For Loop:** If you are familiar with the `for` loop structure, imagine you wished to run the following loop:
            
                :::console
                for read_subset_file in reads_fastq_chunk_a reads_fastq_chunk_b reads_fastq_chunk_c ... reads_fastq_chunk_z
                do
                    ./minimap2.sh $(read_subset_file)
                done
                
            In the example above, we would feed the list of FASTQ files in `~/scaling-up/inputs/` to the variable `$(read_subset_file)` as a list of strings. To express your jobs as a `for` loop in condor, we would instead use the `queue <Var> in <List>` syntax. In the example above, this would be represented as: 
            
                queue read_subset_file in (reads_fastq_chunk_a reads_fastq_chunk_b reads_fastq_chunk_c ... reads_fastq_chunk_z)
    
        !!! success "" 
            **While Loop:** A closer representation to HTCondor's _list of jobs_ structure is the `while` loop. If you are familiar with the `while` loop in bash, you could also consider the set of job submissions to mirror something like:
            
                :::console
                while read read_subset_file;
                do
                    ./minimap2.sh $(read_subset_file)
                done < list_of_fastq.txt
            
            Here we feed the contents of `list_of_fastq.txt`, the list of files in `~/scaling-up/inputs/` to the same `$(read_subset_file)` variable. The `while` loop iterates through each line of `list_of_fastq.txt`, appending the line's value to `$(read_subset_file)`. To express your jobs as a `for` loop in condor, we would instead use the `queue <Var> in <List>` syntax. In the example above, this would be represented as: 
            
                queue read_subset_file from ./list_of_files.txt
        
        For jobs with more than 5 values, we generally recommend using the `queue var from list_of_files.txt` syntax. 

7.  Generate `./test_list_of_fastq.txt` using `head` command

        head -n 3 ./list_of_fastq.txt > ./test_list_of_fastq.txt

7.  Submit your job and monitor its progress.
    
    Submit your test job using `condor_submit`

        :::console
        condor_submit multi_job_minimap.sub

    Monitor the progress of your job using `condor_watch_q`

        :::console
        condor_watch_q
        [OUTPUT OF WATCH Q]

??? success "Always Check Your Test Jobs Worked!"

    Review your `condor_watch_q` output and your files on the Access Point. 

## Submit Multiple Jobs - Scaling to Full Dataset

Now, you are ready to submit the whole workload. 

!!! example "Try It Yourself!"

    You've split your large FASTQ file into multiple read subsets, and you're ready to run `minimap2` on all of them in parallel.

    1. Edit your submit file to use the `queue <var> from <file>` syntax.
    2. Ensure the `arguments`, `transfer_input_files`, `transfer_output_files` fields change with each input.
    3. Ensure your `log`, `error`, and `output` files all include the name of the read subset file being used mapped in this job.
    4. Organize the output files using the correct `transfer_output_remaps` statement
    
    **Before submitting:**

    * Are all your subset filenames listed in `list_of_fastq.txt`?
        * Did you test at least one job successfully?
        * Are you remapping outputs into the `outputs/` folder?

    **Think about what needs to change on your `multi_job_minimap.sub` submit file to submit the full dataset.**

    When ready, submit with:
    ```
    condor_submit minimap2_multi.submit
    ```

    ??? success "Solution - ⚠️ Try to Solve Before Viewing ⚠️"

        **Make sure to change your `queue` statement to use the full dataset.**
        
        Your final submit file, `minimap2_multi.submit`, should look something like this:
            
            +SingularityImage      = "osdf:///ospool/ap40/data/<user.name>/scaling-up/software/minimap2.sif"
            
            executable             = ./minimap2.sh
            arguments              = $(read_subset_file)
            transfer_input_files   = ./input/$(read_subset_file), osdf:///ospool/ap40/data/<user.name>/scaling-up/inputs/reference_genome.fasta
            
            transfer_output_files  = ./$(read_subset_file)_output.sam
            transfer_output_remaps = "$(read_subset_file)_output.sam=output/$(read_subset_file)_output.sam"
            
            output                 = logs/output/job.$(ClusterID).$(ProcID)_$(read_subset_file)_output.out
            error                  = logs/error/job.$(ClusterID).$(ProcID)_$(read_subset_file)_output.err
            log                    = logs/log/job.$(ClusterID).$(ProcID)_$(read_subset_file)_output.log
            
            request_cpus           = 2
            request_disk           = 4 GB
            request_memory         = 4 GB 
            
            queue read_subset_file from ./list_of_fastq.txt

## Checking Your Jobs' Progress

We can use the command `condor_watch_q` to track our job submission. As your jobs progress through the various `job state`, Condor will update the output of `condor_watch_q`.

!!! tip "Pro-Tip: Jobs Holds *Aren't* Always Bad!"
    
    Seeing your jobs in the Held state? Don’t panic! This is often just HTCondor doing its job to protect your workflow.

    Holds can happen for a variety of reasons: missing input files, typos in submit files, or temporary system issues. In many cases, these issues are transient and easily recoverable.

    **To check why your job is held:**

        :::console
        condor_q -held

    **To release your held job after fixing the issue (e.g., typos or missing files):**
    [Not necessarily true. They'll need `condor_qedit` or resubmission]

        :::console
        condor_release <JobID>

    If you're unsure whether to release or dig deeper, or if the hold message is cryptic—we’re here to help! Reach out to the OSG Support team at [support@osg-htc.org](mailto:support@osg-htc.org) with your job ID(s) and a brief description of what you're trying to do. We’re always happy to assist.

    ✅ **Remember: held jobs are a signal, not a failure. Use them to improve and scale your workflows with confidence.**
