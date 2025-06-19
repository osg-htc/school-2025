# Scaling Up Exercise 2.3: Software Differences Between OSPool and the CHTC Pool

The goal of this exercise is to see some differences in the default Linux versions 
and software available in the OSPool and CHTC Pool. We will do this by submitting 
a large batch of jobs that runs a simple shell script, testing for software versions. 

## Create and test a software probe

The following shell script probes for software and returns the version if it is installed:

```bash
#!/bin/sh

host_info(){
hostname
source /etc/os-release
echo $PRETTY_NAME
}

get_version(){
    program=$1
    $program --version > /dev/null 2>&1
    double_dash_rc=$?
    $program -version > /dev/null 2>&1
    single_dash_rc=$?
    which $program > /dev/null 2>&1
    which_rc=$?
    if [ $double_dash_rc -eq 0 ]; then
        $program --version 2>&1
    elif [ $single_dash_rc -eq 0 ]; then
        $program -version 2>&1
    elif [ $which_rc -eq 0 ]; then
        echo "$program installed but could not find version information"
    else
        echo "$program not installed"
    fi
}

host_info
get_version 'R'
get_version 'cmake'
get_version 'python'
get_version 'python3'
```

If there's a specific command line program that your research requires, feel free to add it to the script!
For example, if you wanted to test for the existence and version of `nslookup`, you would add the following to the end
of the script:

``` file
get_version 'nslookup'
```

1. Log into `ap2003.chtc.wisc.edu`. 
1. Create a folder for this exercise
1.  Save the above script as a file named `sw_probe.sh`
1.  Make sure the script can be run: 

		chmod a+x sw_probe.sh

1.  Try running the script in place to make sure it works: 

		./sw_probe.sh

## Surveying the CHTC Pool

Now we will run our shell script on the CHTC pool to survey what resources are available. 

1.  Create a submit file that runs `sw_probe.sh` 100 times
    and uses submit file variables to write different `output`, and `error` files. You 
    might want to separate these into their own folder. 
1.  Submit your job and wait for the results

!!! Tip "Summarizing Results"
	Here's a little bit of shell magic to summarize the results. If you want to 
	compare the output from line 3 across all the jobs (`R`, if you didn't edit
	the above script): 
	
	```
	[user.name@ap2003]$ for f in *.out; do cat $f | head -n 4 | tail -n 1; done | sort | uniq -c
	```

## Surveying the OSPool

1. Copy or recreate the job setup above on `ap40.uw.osg-htc.org`. 
1. Submit the jobs. 
1. Look at the results. Similar or different? 

## Reflection

Were there any surprises about what was available? Knowing what software is likely 
(or not) to be available on a given system can inform how you install or manage 
your software when submitting jobs. CHTC does not have a lot of software available 
by default, so software portability is important. Because the OSPool varies so much, 
containers are a good idea. 