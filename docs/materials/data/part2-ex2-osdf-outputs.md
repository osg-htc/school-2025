---
status: reviewed
---

Data Exercise 2.2: Using OSDF for outputs
=========================================================
This exercise will use a [Minimap2](https://github.com/lh3/minimap2) workflow to
demonstrate the functionality of OSDF for transferring input files to jobs on OSG.

As presented in the Data [lecture](files/osgus25-data.pdf), for submitting multiple jobs that require larger files, OSDF not only reduces the load on the OSPool network
but also keeps a copy of the files in different caching sites, which results in faster transfer of files to the Execution Point. 


Setup
-----
- Please make sure that you are logged into `ap40.uw.osg-htc.org`.
- Create a folder named `minimap2-read` and `cd` into the directory
- Download the required files for this lesson using the following command

        osdf object get /ospool/uc-shared/public/school/2025/minimap2.sif .
        osdf object get /ospool/uc-shared/public/school/2025/Celegans_ref.fa .

!!! note "File Sizes"
    - What are the sizes of the container and the reference genome file?


Prepare files for the job
--------------------------------
Minimap2 compares and aligns large pieces of genetic information, like DNA sequences. This is useful for tasks like identifying similarities between genomes, verifying sequences, or assembling genetic data from scratch. For this exercise, we will perform indexing using the reference genome.


Place files in OSDF
--------------------------------

There are some files we will be using frequently that do not change often. One example of this is the apptainer/singularity container image we will be using for run our minimap2 mappings. The Open Science Data Federation is a data lake accessible to the OSPool with built in caching. The OSDF can significantly improve throughput for jobs by caching files closer to the execution points. 

>[!WARNING]
> The OSDF caches files aggressively. Using files on the OSDF with names that are not unique from previous versions can cause your job to download an incorrect previous version of the data file. We recommend using unique version-controlled names for your files, such as `data_file_04JAN2025_version4.txt` with the data of last update and a version identifier. This ensures your files are correctly called by HTCondor from the OSDF. 

### Copy your data to the OSDF space

OSDF provides a directory for you to store data which can be accessed through the caching servers.
First, you need to move your `minimap2.sif` image. For `ap40.uw.osg-htc.org`, the directory
to use is `/ospool/ap40/data/[USERNAME]/`

Note that files placed in the `/ospool/ap40/data/[USERNAME]/` directory will only be accessible
by your own jobs.

Second, the exercise will use a reference genome file named `Celegans_ref.mmi` to map against the FASTQ files. Copy the `Celegans_ref.mmi` file to your OSDF directory as well.

Considerations
--------------
- Why `minimap2.sif` is placed in the OSDF directory? Is it a big container?

Create a wrapper script and submit jobs 
---------------------------------------

We will need our wrapper script and submit file to use OSDF. Perform the following steps.

1. Copy the following contents and save that as `minimap2_index.sh`

        #!/bin/bash
        minimap2 -x map-ont -d Celegans_ref.mmi Celegans_ref.fa

2. Create `minimap2_index.sub` using either `vim` or `nano`

       ```
        container_image      = "osdf:///ospool/ap40/data/[USERNAME]/minimap2.sif"
    
        executable             = ./minimap2_index.sh
        
        transfer_input_files   = Celegans_ref.fa
    
        transfer_output_files  = Celegans_ref.mmi 
        transfer_output_remaps = "Celegans_ref.mmi = /ospool/ap40/data/[USERNAME]/Celegans_ref.mmi"
        output                 = indexing_step1_$(Cluster)_$(Process).out
        error                  = indexing_step1_$(Cluster)_$(Process).err
        log                    = indexing_step1_$(Cluster)_$(Process).log
        
        request_cpus           = 4
        request_disk           = 5 GB
        request_memory         = 5 GB 
        
        queue 1
       ```
   
> [!IMPORTANT]  
> Notice that we are using the `transfer_output_remaps` attribute in our submit file. By default, HTCondor will transfer outputs to the directory where we submitted our job from. Since we want to transfer the indexed reference genome file `Celegans_ref.mmi` to a specific directory, we can use the `transfer_output_remaps` attribute on our submission script. The syntax of this attribute is:
>  
>   ```transfer_output_remaps = "<file_on_execution_point>=<desired_path_to_file_on_access_point>``` 
>  
> It is also important to note that we are transferring our `Celegans_ref.mmi` to the OSDF directory `/ospool/<ap##>/data/<user.name>/`. We will be reusing our indexed reference genome file for each mapping job in the ScalingUp exercises, we benefit from the caching feature of the OSDF. Therefore, we can direct `transfer_output_remaps` to redirect the `Celegans_ref.mmi` file to our OSDF directory.

   3. Submit your `minimap2_index.sub` job to the OSPool
       ```
      condor_submit minimap2_index.sub
      ```



