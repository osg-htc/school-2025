# Scaling Up Exercise 2.1: Log Into a Local Pool

## Introduction

So far, all of our HTC exercises (and your workloads) have used the Open Science
Pool, accessed through the `ap40` Access Point. But what if you had access to 
another computing resource also? In this section of exercises, we will explore 
similarities and differences between the OSPool, and a campus-focused high throughput computing 
system operated by the Center for High Throughput Computing. 

## Log In

The CHTC Access Point we will be using is called `ap2003.chtc.wisc.edu`. You should be 
able to log into this computer with the same username and login steps as for 
`ap40.uw.osg-htc.org`. 

## Initial Exploration

In the following exercises, we will explore the differences between the OSPool and 
the CHTC pool more deeply. But to start, let's run a few commands to compare. 

### What's in the pool? 

1. Log into `ap2003.chtc.wisc.edu`.
1. Run these commands: 
	``` console
[username@ap2003 ~]$ condor_status | tail
[username@ap2003 ~]$ condor_status -compact | tail -n 15
```
1. How many "slots" are in the pool? How many of each operating system? 
1. In a separate window, log into `ap40.uw.osg-htc.org` and run the same commands. 
1. How many "slots" are in the OSPool? How many of each operating system? 

### Jobs in the queue

1. On `ap2003.chtc.wisc.edu` run `condor_q -all`. 
1. On `ap40.uw.osg-htc.org`, run the same command. 
1. How many jobs are on `ap40` compared to `ap2003`? 
