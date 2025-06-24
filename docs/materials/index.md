# OSG School Materials

## School Overview and Intro

View the slides: [pdf](welcome/files/osgs25-day1-part1-welcome-timc.pdf)

## Intro to HTC and HTCondor Job Execution

### Intro to HTC Slides

Intro to HTC: [pdf](htcondor/files/osgschool25-htc-intro.pdf)

Worksheet: [pdf](htcondor/files/HTC-List-Of-Jobs.pdf) or [Google Drive](https://docs.google.com/presentation/d/1DnJMOuw0YjuVg70vCvafXlJUm4k5Ovv7Zd1SJbfk6hU/edit?usp=sharing)

### Intro to HTCondor Slides

View the slides: [pdf](htcondor/files/osgus25-htc-htcondor.pdf)
<!-- [PowerPoint](htcondor/files/osgus22-htc-htcondor.pptx)) -->

### Intro Exercises 1: Running and Viewing Simple Jobs (Strongly Recommended)

- [Exercise 1.1: Log in to the local submit machine and look around](htcondor/part1-ex1-login.md)
- [Exercise 1.2: Experiment with HTCondor commands](htcondor/part1-ex2-commands.md)
- [Exercise 1.3: Run jobs!](htcondor/part1-ex3-jobs.md)
- [Exercise 1.4: Read and interpret log files](htcondor/part1-ex4-logs.md)
- [Exercise 1.5: Determining Resource Needs](htcondor/part1-ex5-request.md)
- [Exercise 1.6: Remove jobs from the queue](htcondor/part1-ex6-remove.md)

### Bonus Exercises: Job Attributes and Handling

- [Bonus Exercise 1.7: Explore `condor_q`](htcondor/part1-ex7-queue.md)
- [Bonus Exercise 1.8: Explore `condor_status`](htcondor/part1-ex8-status.md)

## Intro to HTCondor Multiple Job Execution

View the Slides: [pdf](htcondor/files/osgus25-htc-htcondor-multiple-jobs.pdf)

### Intro Exercises 2: Running Many HTC Jobs (Strongly Recommended)

- [Exercise 2.1: Work with input and output files](htcondor/part2-ex1-files.md)
- [Exercise 2.2: Use `queue N`, `$(Cluster)`, and `$(Process)`](htcondor/part2-ex2-queue-n.md)
- [Exercise 2.3: Use `queue from` with custom variables](htcondor/part2-ex3-queue-from.md)
- [Bonus Exercise 2.4: Use `queue matching` with a custom variable](htcondor/part2-ex4-queue-matching.md)

## The Open Science Pool (OSPool)

View the slides: [Coming soon]
<!-- View the slides: [pdf](osg/files/osgs24-day2-part1-osg-timc.pdf) -->

### OSPool Exercises: Researching the OSPool (Strongly Recommended)

- [Exercise 1.1: Where Do Jobs Run?](ospool/part1-ex1-where-run.md)
- [Exercise 1.2: How Much Can I Get?](ospool/part1-ex2-capacity.md)
- [Exercise 1.3: How Does Capacity Change?](ospool/part1-ex3-dynamic-capacity.md)
- [Exercise 1.4: What Is In an Execution Point?](ospool/part1-ex4-ep-sandbox.md)
- [Bonus Exercise 1.5: Viewing OSPool Information](ospool/part1-ex5-ospool-views.md)

## Troubleshooting

View the Slides: [pdf](troubleshooting/files/OSGUS2025_troubleshooting.pdf) [ppt](troubleshooting/files/OSGUS2025_troubleshooting.pptx)

### Troubleshooting Exercises: 

- [Exercise 1.1: Troubleshooting Jobs](troubleshooting/part1-ex1-troubleshooting.md)
- [Exercise 1.2: Job Retry](troubleshooting/part1-ex2-job-retry.md)

## Software

View the Slides: [pdf](software/files/osgs25-software.pdf), [pptx](software/files/osgs25-software.pptx)

### Software Exercises 1: Exploring Containers

- [Exercise 1.1: Run and Explore Apptainer Containers](software/part1-ex1-run-apptainer.md)
- [Exercise 1.2: Use Apptainer Containers in OSPool Jobs](software/part1-ex2-apptainer-jobs.md)
- [Exercise 1.3: Use Docker Containers in OSPool Jobs](software/part1-ex3-docker-jobs.md)
- [Exercise 1.4: Build, Test, and Deploy an Apptainer Container](software/part1-ex4-apptainer-build.md)
- [Exercise 1.5: Choose Software Options](software/part1-ex5-pick-an-option.md)

### Software Exercises 2: Preparing Scripts
- [Exercise 2.1: Build an HTC-Friendly Executable](software/part2-ex1-build-executable.md)

### Software Exercises 3: Container Examples (Optional)

- [Exercise 3.1: Create an Apptainer Definition Files](software/part3-ex1-apptainer-recipes.md)
- [Exercise 3.2: Build Your Own Docker Container](software/part3-ex2-docker-build.md)

### Software Exercises 4: Exploring Compiled Software (Optional)

- [Exercise 4.1: Download and Use Compiled Software](software/part4-ex1-download.md)
- [Exercise 4.2: Use a Wrapper Script To Run Software](software/part4-ex2-wrapper.md)
- [Exercise 4.3: Using Arguments With Wrapper Scripts](software/part4-ex3-arguments.md)

### Software Exercises 5: Compiled Software Examples (Optional)

- [Exercise 5.1: Compiling a Research Software](software/part5-ex1-prepackaged.md)
- [Exercise 5.2: Compiling Python and Running Jobs](software/part5-ex2-python.md)
- [Exercise 5.3: Using Conda Environments](software/part5-ex3-conda.md)
- [Exercise 5.4: Compiling and Running a Simple Code](software/part5-ex4-compiling.md)

## Data

View the Slides: [Coming soon]
<!-- View the slides: [pdf](data/files/osgus24-data.pdf) -->

### Data Exercises 1: HTCondor File Transfer (Strongly Recommended)

- [Exercise 1.1: Understanding a job's data needs](data/part1-ex1-data-needs.md)
- [Exercise 1.2: transfer\_input\_files, transfer\_output\_files, and remaps](data/part1-ex2-file-transfer.md)
- [Exercise 1.3: Splitting input](data/part1-ex3-blast-split.md)

### Data Exercises 2: Using OSDF (Strongly Recommended)

- [Exercise 2.1: OSDF for inputs](data/part2-ex1-transfer-file-using-osdf.md)
- [Exercise 2.2: OSDF for outputs](data/part2-ex2-osdf-outputs.md)

## Scaling Up

View the Slides: [Coming soon]
<!-- View the slides: [pptx](scaling/files/osgschool24-learning-scaling.pptx) -->

### Scaling Up Exercises

-   [Exercise 1.1: Organizing HTC workloads](scaling/part1-ex1-organization.md)
-   [Exercise 1.2: Composing Your Jobs](scaling/part2-ex1-composing-the-job.md)
-   [Exercise 1.2: Investigating Job Attributes](scaling/part1-ex2-job-attributes.md)
-   [Exercise 1.3: Getting Job Information from Log Files](scaling/part1-ex3-log-files.md)

## Workflows with DAGMan

View the Slides: [Coming soon]
<!-- View the slides: [pptx](workflows/files/osg24-dagman.pptx) -->

### DAGMan Exercises 1

- [Exercise 1.1: Coordinating set of jobs: A simple DAG](workflows/part1-ex1-simple-dag.md)
- [Exercise 1.2: A brief detour through the Mandelbrot set](workflows/part1-ex2-mandelbrot.md)
- [Exercise 1.3: A more complex DAG](workflows/part1-ex3-complex-dag.md)
- [Exercise 1.4: Handling jobs that fail with DAGMan](workflows/part1-ex4-failed-dag.md)
- [Exercise 1.5: Workflow Challenges](workflows/part1-ex5-challenges.md)

## Extra Topics

<!-- BEGIN EXTRA TOPICS THAT ARE NOT READY YET


### Containers (and GPUs?)

View the slides
([PDF](gpus/files/osgvsp21-gpus-containers.pdf),
[PowerPoint](gpus/files/osgvsp21-gpus-containers.pptx))

- [Exercise 1.1: Containers Overview](gpus/part1-ex1-containers-overview.md)
- [Exercise 1.2: Running a CPU job](gpus/part1-ex2-cpu-jobs.md)
- [Exercise 1.3: Running a GPU job](gpus/part1-ex3-gpu-jobs.md)

END EXTRA TOPICS THAT ARE NOT READY YET -->

### Machine Learning

View the Slides: [Coming soon]
<!-- View the slides: [pdf](ml/Throughput-ML-iross-OSGUS24.pdf) -->

### Self-checkpointing for long-running jobs

View the Slides: [Coming soon]
<!-- View the slides: [pdf](checkpoint/files/OSGUS2024_checkpointing.pdf) [ppt](checkpoint/files/OSGUS2024_checkpointing.pptx) -->

-   [Exercise 1.1: Trying out self-checkpointing](checkpoint/part1-ex1-checkpointing.md)

### Special Environments

View the slides: [Coming soon]

### Special Environments Exercises 1

- [Exercise 1.1: GPUs](special/part1-ex1-gpus.md)

### Introduction to Research Computing Facilitation

View the slides: [Coming soon]

## Final Talks

*   Philosophy: [Slides coming soon]
*   Final thoughts: [Slides coming soon]
*   Forward (Tim’s final talk):
    [PDF](final/files/osgs24-day5-part9-forward-timc.pdf)
