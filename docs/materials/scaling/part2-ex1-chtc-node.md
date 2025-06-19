---
status: testing
---

# Scaling Up Exercise 2.1: Log Into a Local Pool

## Introduction

So far, all of our HTC exercises (and your workloads) have used the Open Science
Pool, accessed through the `ap40` Access Point. But what if you had access to 
another computing resource also? In this section of exercises, we will explore 
similarities and differences between the OSPool, and a campus-focused high throughput computing 
system operated by the Center for High Throughput Computing. 

## Log In



### Set up ssh key

In later exercises, it might be useful to transfer files from the CHTC Access Point 
(`ap2003`) to the OSPool Access Point (`ap40`). To facilitate this process, follow 
the steps below to set up an ssh key that will allow you to connect the two. 

1. Make sure you are logged into `ap2003.chtc.wisc.edu`
1. Run this command
		$ ssh-keygen
1. Download the generated key file to your desktop. 
1. Log into the [COmanage Registry](https://registry.cilogon.org/).
1. Click on your profile and add the key. 

## Initial exploration

`condor_status`