---
status: testing
---

<style type="text/css"> pre em { font-style: normal; background-color: yellow; } pre strong { font-style: normal; font-weight: bold; color: \#008; } </style>

HTC Exercise 1.1: Log In and Look Around
===========================================

Background
----------

There are different High Throughput Computing (HTC) systems at universities, government facilities, and other institutions around the world, and they may have different user experiences. For example, some systems have dedicated resources (which means your job will be guaranteed a certain amount of resources/time to complete), while other systems have opportunistic, backfill resources (which means your job can take advantage of some resources, but those resources could be removed at any time). Other systems have a mix of dedicated and opportunistic resources. 

Durring the OSG School, you will practice on the "[OSG's Open Science Pool (OSPool)](https://osg-htc.org/services/open_science_pool.html)". The OSPool provides researchers with **opportunitistic resources and the ability to run many smaller and shorter jobs silmnulatinously**. The OSPool is composed of approximately 60,000+ cores and dozens of different GPUs. 

Exercise Goal 
---

The goal of this first exercise is to log in to the OSPool access point and look around a little bit, which will take only a few minutes. 

**If you have trouble getting SSH access to the access point, ask the instructors right away! Gaining access is critical for all remaining exercises.**

Logging In
----------

You will login to OSPool access pool, `ap40.uw.osg-htc.org`, using the username assigned to you. 

To log in, use a [Secure Shell](http://en.wikipedia.org/wiki/Secure_Shell) (SSH) client.

-   On Windows, you can use Terminal (Powershell or WSL), [MobaXTerm](https://mobaxterm.mobatek.net/), [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/), VSCode, or any other SSH client.
-   On Mac or Linux, you may use the Terminal application or any other SSH client.

To log in, run the `ssh` command as shown below, replacing `<USERNAME>` with your username. Do not include the brackets (`<>`).

```
ssh <USERNAME>@ap40.uw.osg-htc.org
```

**If you need help finding or using an SSH client, ask the instructors for help right away**!

Running Commands
----------------

In the exercises, we will show commands that you are supposed to type or copy into the command line, like this:

``` console
[username@ap40 ~]$ hostname
ospool-ap2140
```

!!! note
    In the first line of the example above, the `[username@ap40 ~]$` part is meant to show the Linux command-line prompt.
    You do not type this part! So in the example above, the command that you type at your own prompt is just the eight characters `hostname`. The second line of the example, without the prompt, shows the output of the command; you do not type this part,    either.

Here are a few other commands that you can try (the examples below do not show the output from each command):

``` console
[username@ap40 ~]$ whoami
[username@ap40 ~]$ date
[username@ap40 ~]$ uname -a
```

Try typing into the command line as many of the commands as you can.
Copy-and-paste is fine, but **you will learn more if you take the time to type each command yourself.**

Organizing Your Workspace
-------------------------

You will be doing many different exercises over the next few days, many of them on this access point. Each exercise may use many files, once finished. To avoid confusion, it is useful to create a separate directory for each exercise. Additionally, directories that contain numerous files can be inefficient. This number varies, depending on the file system type.

To create a directory, use the `mkdir` command. We can change directories with the `cd` command, as shown below.

``` console
[username@ap40 ~]$ mkdir intro-1.1-login
[username@ap40 ~]$ cd intro-1.1-login
```

To return to the directory above the `intro-1.1-login` directory, use the `cd` command with two dots (`..`).

``` console
[username@ap40 intro-1.1-login]$ cd ..
[username@ap40 ~]$
```

### The meaning of (`..`), (`.`), and (`~`)

Sometimes, you'll see (`..`), (`.`), and (`~`) in commands and paths. These symbols represent different paths in the system.

* Two dots (`..`) represent the directory above the current directory.
* One dot (`.`) represents the current directory.
* The tilde symbol (`~`) represents your home directory (`/home/username`).

Showing the Version of HTCondor
-------------------------------

HTCondor is installed on this server. But what version? You can ask HTCondor itself:

``` console
[user@ap40 ~]$ condor_version
$CondorVersion: 24.7.3 2025-04-17 BuildID: 802763 PackageID: 24.7.3-0.802763 GitSHA: bb5d9294 RC $
$CondorPlatform: x86_64_AlmaLinux9 $
```

Reference Materials
-------------------

Here are a few links to reference materials that will be useful on your journey in research computing!

- [Software Carpentry Unix Shell Lesson](https://eharstad.github.io/shell-novice/); This lesson is a good place to start learning about the Unix shell, including how to navigate directories, create files, and write simple scripts.
- [HTCondor manuals](https://htcondor.readthedocs.io/en/latest/); we recommend reading the manual corresponding to the version of HTCondor that you use. That link points to the latest version of the manual, but you can switch versions using the toggle in the lower left corner of that page.
