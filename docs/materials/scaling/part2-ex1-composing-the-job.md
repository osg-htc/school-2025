# Scaling Up Our Workload 

High-throughput computing (HTC) allows us to efficiently scale this analysis by distributing jobs across many computing resources. In this lesson, you'll learn how to structure and submit a read mapping workflow using the OSPool and `minimap2`. This includes adapting your executable script and submit file to dynamically handle many input files in parallel.

In our previous exercise, [Scaling-Up Exercise 1 Part 1](../part1-ex1-organization), you prepared your directory structure and organized your data for distributed processing. In this section, we’ll move forward with composing and testing your HTC jobs to take full advantage of the OSPool's capacity.

!!! halt "**Halt!** Do not proceed if you haven't completed the [Scaling-Up Exercise 1 Part 1](../part1-ex1-organization)"
    
    This is part two of our Scaling Up Exercise 1 set and should **only** be completed after you've successfully completed [Scaling-Up Exercise 1 Part 1](../part1-ex1-organization). 

## Log into an OSPool Access Point

Make sure you are logged into `ap40.uw.osg-htc.org`. 

## Composing Your Job

### Adapting the Executable
Now that we have our data partitioned into independent subsets to be mapped in parallel, we can work on adapting our executable for use on the OSPool. We will start with the following template executable file, which is also found in your project directory under `~/scaling-up/minimap2.sh`.

    :::console
    #!/bin/bash
    # Use minimap2 to map the basecalled reads to the reference genome
    minimap2 -ax map-ont reference_genome.fasta reads.fastq > output.sam

| Command Segment | `minimap2`                             | `-ax map-ont`                                                                      | `reference_genome.fasta`               | `reads.fastq`                                 | `>`                                         | `output.sam`                          |
|-----------------|----------------------------------------|----------------------------------------------------------------------------------|--------------------------------------|---------------------------------------------|--------------------------------------------|-------------------------------------|
| **Meaning**         | The program we'll run to map our reads | Specifies the type of reads we're using <br>(Oxford Nanopore Technologies reads) | The input reference we're mapping to | The reads we are mapping against our genome | redirects the output of minimap2 to a file | The output file of our mapping step |

!!! halt "Time-Out! **Think about how you would adapt this executable template for HTC**"

    If we want to map each one of our reads subsets against the reference genome, think about the following questions

        * What parts of the command will change with each job?
        * What parts of the command will stay the same?

Let's start by editing our template executable file! In our executable there's two main segments of the `minimap2` command that will be changed: The input `reads.fastq` file and the output `output.sam` file. 

!!! warning "Thinking Ahead Before Errors! **Renaming our output files**"

    What do you think would happen if we do keep the output file on our executable as `output.sam`?

1. Modify the executable to accept the name of our input `reads.fastq` subsets as an argument.

        :::console
        #!/bin/bash
        {++reads_subset_file="$1"++}
        # Use minimap2 to map the basecalled reads to the reference genome
        minimap2 -ax map-ont reference_genome.fasta {==$(reads_subset_file)==} > output.sam

2. Modify the executable use the name of our input reads subset file (`$reads_subset_file`) as the prefix of our output file.

        :::console
        #!/bin/bash
        reads_subset_file="$1"
        # Use minimap2 to map the basecalled reads to the reference genome
        minimap2 -ax map-ont reference_genome.fasta "$(reads_subset_file)" > {=="$(reads_subset_file)_output.sam"==}

### Generating the List of Jobs
Next, we need to generate a list of jobs for HTCondor to run. In previous exercises, we've used the `queue` statements such as `queue <num>` and `queue <variable> matching *.txt`. For our exercise, we will use the `queue <var> from <list>` submission strategy. 

!!! pro-tip "Think Ahead!" 
    What values should we pass to HTCondor to scale our `minimap2` workflow up? 

1. Move to your `~/scaling-up/inputs/` directory
        
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

!!! pro-tip "Pro-Tip: Always Test Your Workflows!"

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

Now, you are ready to submit the whole workload. We can think of our multi-job submission as a sort of `for` or `while` loop in bash. If you are familiar with the `for` loop structure, imagine you wished to run the following loop:

    :::console
    for fastq_read_subset_file in reads_fastq_chunk_a reads_fastq_chunk_b reads_fastq_chunk_c ... reads_fastq_chunk_
    do
        ./minimap2.sh $(fastq_read_subset_file)
    done
    
In the example above, we would feed the list of FASTQ files in `~/scaling-up/inputs/` to the variable $(fastq_read_subset_file) as a list of strings. A closer representation to HTCondor's _list of jobs_ structure is the `while` loop. If you are familiar with the `while` loop in bash, you could also consider the set of job submissions to mirror something like:

    :::console
    while read fastq_read_subset_file;
    do
        ./minimap2.sh $(fastq_read_subset_file)
    done < list_of_fastq.txt

Here we feed the contents of `list_of_fastq.txt`, the list of files in `~/scaling-up/inputs/` to the same `$(fastq_read_subset_file)` variable. The `while` loop iterates through each line of `list_of_fastq.txt`, appending the line's value to `$(fastq_read_subset_file)`. 

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

    When ready, submit with:
    ```
    condor_submit minimap2_multi.submit
    ```

    ??? success "Solution Set - ⚠️ Try to Solve Before Viewing ⚠️"

        Your final submit file, `minimap2_multi.submit`, should look something like this:
            
            +SingularityImage      = "osdf:///ospool/ap40/data/<user.name>/scaling-up/software/minimap2.sif"
            
            executable             = ./minimap2.sh
            {==arguments              = $(read_subset_file)==}
            transfer_input_files   = {==./input/$(read_subset_file)==}, osdf:///ospool/ap40/data/<user.name>/scaling-up/inputs/reference_genome.fasta
            
            transfer_output_files  = {==./$(read_subset_file)_output.sam==}
            transfer_output_remaps = {=="$(read_subset_file)==}_output.sam=output/{==$(read_subset_file)==}_output.sam"
            
            output                 = logs/output/job.$(ClusterID).$(ProcID){==_$(read_subset_file)==}_output.out
            error                  = logs/error/job.$(ClusterID).$(ProcID){==_$(read_subset_file)==}_output.err
            log                    = logs/log/job.$(ClusterID).$(ProcID){==_$(read_subset_file)==}_output.log
            
            request_cpus           = 2
            request_disk           = 4 GB
            request_memory         = 4 GB 
            
            queue {==read_subset_file from ./list_of_fastq.txt==}

## Checking Your Jobs' Progress

We can use the command `condor_watch_q` to track our job submission. As your jobs progress through the various `job state`, Condor will update the output of `condor_watch_q`.

!!! pro-tip "Pro-Tip: Jobs Holds *Aren't* Always Bad!"
    
    Seeing your jobs in the Held state? Don’t panic! This is often just HTCondor doing its job to protect your workflow.

    Holds can happen for a variety of reasons: missing input files, typos in submit files, or temporary system issues. In many cases, these issues are transient and easily recoverable.

    **To check why your job is held:**

        :::console
        condor_q -held

    **To release your held job after fixing the issue (e.g., typos or missing files):**

        :::console
        condor_release <JobID>

    If you're unsure whether to release or dig deeper, or if the hold message is cryptic—we’re here to help! Reach out to the OSG Support team at [support@osg-htc.org](mailto:support@osg-htc.org) with your job ID(s) and a brief description of what you're trying to do. We’re always happy to assist.

    ✅ **Remember: held jobs are a signal, not a failure. Use them to improve and scale your workflows with confidence.**

!!! halt "FOR STAFF REVIEWERS: IGNORE BELOW"





??? example "Challange: Scaling Beyond by Generalizing Our Scripts"

    You've successfully mapped your C. elegans reads to a reference genome using HTCondor on the OSPool, but what if your project includes other species, new reference versions, or additional sequencing runs? Let’s take it a step further and make your workflow modular, reproducible, and interchangeable!
    
    **Generalize Your Workflow**
    
    Let's start by generalizing our executable file `minimap2.sh`. 
    
    1. Review your `minimap2.sh` file below
    
            :::console
            #!/bin/bash
            reads_subset_file = "$1"
            # Use minimap2 to map the basecalled reads to the reference genome
            minimap2 -ax map-ont reference_genome.fasta "$(reads_subset_file)" > "$(reads_subset_file)_output.sam"
    
        !!! example "Try It Yourself!"
        
            Ask yourself the following guiding questions while generalizing this script:
        
            1. Can we use arguments to represent additional parts of the `minimap2` command?
            2. Are all our files represented by a variable or are some hard-coded into the script?
            3. What about our command flags?
    
            ??? success "Solution Set - ⚠️ Try to Solve Before Viewing ⚠️"
        
                Your final executable file, `minimap2.sh`, should look something like this:
                    
                    :::console
                    #!/bin/bash
                    run_options="$1"
                    reference_file="$2"
                    reads_subset_file="$3"
                    output_file_suffix="$4"
                    # Use minimap2 to map the basecalled reads to the reference genome
                    minimap2 $run_options "$reference_file" "$reads_subset_file" > "${reads_subset_file}${output_file_suffix}"
          
                We now have a `minimap2.sh` executable that can accept four (4) arguments representing each segment of the command. This allows us to quickly and easily adapt our minimap run to different presets, files, or run modes. 
    
       2. Review your `minimap2.sub` file below
    
               :::console
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

        !!! example "Try It Yourself!"
        
               Ask yourself the following guiding questions while generalizing this script:
        
               1. Can we use dynamically assign values to any of the Job ClassAds?
                   a. Which Job ClassAds are static (won't change much between submissions)?
                   b. Which Job ClassAds are dynamic (change between submissions)?
               2. Are all our files represented by a variable or are some hard-coded into the script?
                   a. Which job files **will not** change between submissions?
               3. What about our arguments for the executable?
    
            ??? success "Solution Set - ⚠️ Try to Solve Before Viewing ⚠️"
    
                   Your final submit file, `minimap2.sub`, should look something like this:
    
                       +SingularityImage      = "osdf:///ospool/ap40/data/<user.name>/scaling-up/software/minimap2.sif"
                        executable             = ./minimap2.sh
                       "one 'two with spaces' 3"
                       arguments              = "'-ax map-ont' reference_genome.fasta $(read_subset_file) _output.sam"
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
    
                   We can now pass through our arguments to the executable that we've generalized.
                
                   !!! pro-tip "Pro-Tip: Spacing in HTCondor's Argument ClassAd"
    
                       HTCondor's `Arguments` ClassAd is interpreted as a space-delimited set of values. This means we need to wrap arguments that contain spaces in them (for example `-ax map-ont`) in single-quotas (`'-ax map-ont'`). We also need to wrap the full set of arguments in double quotes ("list of arguments"). For more information on advanced argument usage, please refer to the [condor_submit Manual Page on ReadTheDocs](https://htcondor.readthedocs.io/en/latest/man-pages/condor_submit.html#arguments)
    
