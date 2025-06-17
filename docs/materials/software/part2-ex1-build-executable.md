---
status: in progress
---

<style type="text/css"> pre em { font-style: normal; background-color: yellow; } pre strong { font-style: normal; font-weight: bold; color: \#008; } </style>

Software Exercise 2.1 Build an HTC-Friendly Executable
============================================================

**Objective**: Modify an existing script to include arguments and headers. 

**Why learn this?**: A little bit of preparation can make it easier to reuse the 
same script over and over to run many jobs. 

Setup
-------

1. Download the materials for this exercise using

		:::console
		$ osdf object get /ospool/uc-shared/public/school/2025/dev/contexts.tar.gz
		$ tar -xzf contexts.tar.gz

1. There are two scripts included in the materials: `word-variations.py` and `word-contexts.py`.
The first one will identify "trivial" variations (capitalization, punctuation) on the provided word.
For example:

		:::console
		$ ./word-variations.py Alice_in_Wonderland.txt Dodo

	The second one expands the detection of the provided word to report an additional word before
	and after the provided word, for example:

		:::console
		$ ./word-contexts.py Alice_in_Wonderland.txt Dodo

1. We want to use these two scripts to find all the contexts where a word was used, regardless of
the "trivial" variations. To do so, we first save the results of the `word-variations.py` command
to a file, then use the contents of that file with the `word-contexts.py` command (also saving the
output to file):

		:::console
		$ ./word-variations.py Alice_in_Wonderland.txt Dodo > variations.Dodo.txt
		$ ./word-contexts.py Alice_in_Wonderland.txt variations.Dodo.txt > contexts.Dodo.txt

Add a Header
----------

1. To create a basic script, you can put the above commands into a file 
called `get_contexts.sh`. To make it clear what language we expect to use to 
run the script, we will add the following header on the first line: ``#!/bin/bash`

		:::file
		#!/bin/bash
		
		./word-variations.py Alice_in_Wonderland.txt Dodo > variations.Dodo.txt
		./word-contexts.py Alice_in_Wonderland.txt variations.Dodo.txt > contexts.Dodo.txt


	The "header" of `#!/bin/bash` will tell the computer that this is a bash shell script 
	and can be run in the same way that  you would run individual commands on the command line. 
	We use `/bin/bash` instead of just `bash` because that is the full path to the `bash` 
	software file.

    !!! note "Other languages"
		We can use the same principle for any scripting language. For example, the header for a Python script 
		could be either `#!/usr/bin/python3` or `#!/usr/bin/env python3`. Similar logic works 
		for perl, R, julia and other scripting languages. 

1. Can you now run the script? 

		:::console
		$ ./get_contexts.sh

1. This gives "permission denied." Let's add executable permissions to the script 
and try again: 

		:::console
		$ chmod +x get_contexts.sh
		$ ./get_contexts.sh

Incorporate Arguments
----------

Let's say that you want to find the contexts for every occurrence of a character's name.
Can you imagine trying to run this script for each name? It would be tedious
to edit it for each one, especially since the name is used in multiple places in the script. 
And if you want to use this script to analyze other texts, you'd also have to change the filename.

Instead of manually editing the `get_contexts.sh` for each time we want to run a different analysis,
we should add arguments to the script to make it easy to reuse. 
Any information in a script or executable that is going to change or vary across 
jobs or analyses should likely be turned into an argument that is specified on the command line. 

1. In our example above, which pieces of the script are likely to change or vary? 

1. The name of the input file (`Alice_in_Wonderland.txt`) and the name to detect (`Dodo`) should 
be turned into arguments. Can you envision what our script should look like if we ran it 
with input arguments? 

1. Let's say we want to be able to run the following command: 

		:::console
		$ ./get_contexts.sh Alice_in_Wonderland.txt Dodo

	In order to get arguments from the command line into the script, you have 
	to use special variables in the script. In bash, these are `$1` (for the first 
	argument), `$2` (for the second argument) and so on. Try to figure out where 
	these should go in our `get_contexts.sh` script. 

    !!! note "Other Languages"
		Each language is going to have its own syntax for reading command line 
		arguments into the script. In Python, `sys.argv` is a basic method, and more 
		advanced libraries like `argparse` can be used. In R, the `commandArgs()` function 
		can do this. Google "command line arguments in ______" to find the right 
		syntax for your language of choice!

1. A first pass at adding arguments might look like this: 

		:::file
		#!/bin/bash
		
		./word-variations.py $1 $2 > variations.$2.txt
		./word-contexts.py $1 variations.$2.txt > contexts.$2.txt
		
	Try running it as described above. Does it work? 

1. While we now have arguments, we have lost some of the readability of our script. The 
numbers `$1` and `$2` are not very meaningful in themselves! Let's rewrite the script to 
assign the arguments to meaningful variable names: 

		:::file
		#!/bin/bash
		
		FILENAME="${1}"
		TARGET_WORD="${2}"

		./word-variations.py ${FILENAME} ${TARGET_WORD} > variations.${TARGET_WORD}.txt
		./word-contexts.py ${FILENAME} variations.${TARGET_WORD}.txt > contexts.${TARGET_WORD}.txt

    !!! note "Why curly brackets?"
        You'll notice above that we started using curly brackets around our variables. 
        While you technically don't need them (`$TARGET_WORD` would also be fine), using 
        them makes the name of the variable (compared to other text) completely clear. 
        This is especially useful when combining variables with underscores, and in the
		case of specifying the numerical arguments (`$1`, `$2`, etc.) the curly braces
		allow you to go above a value of `9` without error.

1. There is one final place where we could optimize this script. The output filename
`variations.${TARGET_WORD}.txt` is used in multiple places in the script. If we decide we
want to change the naming convention of the output file, we'd have to remember to edit 
all the locations where that name is used, which is asking for typos. Instead, we 
should use a variable for referring to the output filename, so that we only ever need to change
one spot in the file. We can do a similar thing for the second output file.
After making these changes, the script will look like this: 

		:::file
		#!/bin/bash
		
		FILENAME="${1}"
		TARGET_WORD="${2}"
		VARIATIONS_OUTPUT="variations.${TARGET_WORD}.txt"
		CONTEXTS_OUTPUT="contexts.${TARGET_WORD}.txt"

		./word-variations.py ${FILENAME} ${TARGET_WORD} > ${VARIATIONS_OUTPUT}
		./word-contexts.py ${FILENAME} ${VARIATIONS_OUTPUT} > ${CONTEXTS_OUTPUT}

	You may consider this optimization to be optimal and that's fine. But for longer scripts,
	being able to change "customizable" values in a single place helps prevent inconsistencies
	when making edits. Furthermore, this structure makes it easier to add additional arguments
	to the script. For example, if we wanted to always define custom names for the output files
	we could change the definitions at the top of script to read in the third and fourth arguments
	as the value, i.e.,

		:::file
		#!/bin/bash
		
		FILENAME="${1}"
		TARGET_WORD="${2}"
		VARIATIONS_OUTPUT="${3}"
		CONTEXTS_OUTPUT="${4}"

		./word-variations.py ${FILENAME} ${TARGET_WORD} > ${VARIATIONS_OUTPUT}
		./word-contexts.py ${FILENAME} ${VARIATIONS_OUTPUT} > ${CONTEXTS_OUTPUT}

Your Work
----------

1. Are you using a scripting language where you could add a header to your main script? 
If so, what should it be? 

1. What items in your main code or commands are changing? Do you need to add arguments 
to your code? 
