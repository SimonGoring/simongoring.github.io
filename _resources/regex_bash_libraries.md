---
title: Regex to the Rescue - Reinstalling packages
concept: Using regular expressions to ease the pain of installing packages for a new project.
res_class: Development
layout: page
---

<div style="border-width:1px;border-style:solid;padding:5px;background-color:#cccccc;margin-bottom:15px;">
<strong>TLDR</strong> This post describes a bash script that can be run in an R project directory. The script, when run automatically installs any packages called within that project. <a href="https://gist.github.com/SimonGoring/fe3b0bf6b4cb9b8f0a2aa6cdce10bb29">The bash script is available in my GitHub Gists</a>.
</div>


Sometimes, when you update to a new version of R, download a project from GitHub or copy a new project from a colleague it's possible that multiple files may load or require R packages that you don't have currently installed.

This can be a real pain in the neck the first time you try to run the various scripts.  If things have been set up well it's often a matter of looking into a single file (perhaps a `setup` file), but in other cases it requires itteratively running the various scripts and installing things package by package.

I've encountered this often enough that I decided to to [write a `bash` script for linux (and MacOS)](https://gist.github.com/SimonGoring/fe3b0bf6b4cb9b8f0a2aa6cdce10bb29) that could search through the various scripts return the full list of packages and then install them one by one.

## Finding R package calls

How do people call packages in R?  This is a pretty good question in general.  There are several patterns we need to check:

```R
library(ggplot2)
library(ggplot2, verbose = TRUE)
library(ggplot2, stringr)
library("ggplot2")
library("ggplot2", "yarrr")
require(stringr)
#' @import fields
#' @importFrom neotoma compile_taxa
neotoma::get_dataset()
dataset %>% dplyr::filter(value > 10) %>% DT::datatable() 
```

Each of these is a valid way to load packages within R.  My preference is to load packages one at a time using `library()` since `library()` returns an error on execution (rather than waiting until a function is called), and since this makes it easier to look through a long list of functions.  Regardless, these multiple methods of loading packages (whether within package definitions or inline) make up the valid set of library loading within R.  So, if we want to capture them we need to be able to figure out how to express them in my favorite tool: regular expressions.

**Caveat**: There are a number of options for the `library()` command, and people might call `library()` any number of ways.  While this is not an exhaustive list, this reflects the way that *I* call packages for the most part, and this is intended (in part) to also serve as a teaching tool for the use of regular expressions and the use of bash scripts.

## Defining our tools

<img src="../images/regex_flow_bash.svg" height="400">

<strong>Figure 1</strong>. A model program flow for the intended script.  <a href="https://thenounproject.com/search/?q=cloud%20computing&i=1495288">Cloud Icon Created by Yo! Baba from the Noun Project</a>.

This workflow is designed to work as a [`bash` script](https://help.ubuntu.com/community/Beginners/BashScripting) within a linux terminal.  I am using Ubuntu 18.04, but it should work on a MacOS terminal as well.  I wrote it to be used as part of a workflow where I could do something like this:

```bash
git clone somegitrepowithRcode
bash installRpkg.sh -i
```

And then run [RStudio](https://www.rstudio.com/) or an editor of my choice (I've been using Atom more lately).

If I wanted to go further with the commandline I could [edit my `.bashrc` file to add an `alias`](http://www.public.iastate.edu/~akmitra/aero361/design_web/AshWWW/labs/bash.html).  For now we'll work on building the regular expressions and then putting them into a bash file.

## Using `sed`

I use [the program `sed`](https://www.gnu.org/software/sed/manual/sed.html) to perform my regular expression matching.  I use `sed` rather than `grep` because `sed` is specifically designed to edit streams of text (`sed` comes from the contraction of String EDitor).  The script will be processing lines of code and returning text to an array, directly interacting with a stream of text.  `sed` also gives us some more tools to work with, and for this project I will be using one particular flag with `sed`:

```bash
sed -n pattern source
```

The `-n` flag tells `sed` not to print intermediate results to the screen, basically, without the `-n` flag `sed` will print all of the `source` to the screen and then print out all the matches.  When we actually get to building the pattern we will want to do something a bit special.  We don't want to return the whole match, we want to generate a regular expression query that results in a *substitution*, so that our match to `library(ggplot2)` return `ggplot2` only.  That way we will get a list of packages.

### The Pattern

The general style for `sed` substitutions is `options/match/substitution/options`. To undertake substitution in `sed` you need to start with the option `s/`.  Our assumption is that each call to a package will occur only once per line of code, but for the `neotoma::get_dataset()` it should be clear that people can call nested functions or string multiple functions on a single line using `%>%` pipes.  Because of this we need to implement a *global* search for that pattern.  To do this we use a terminal "global" option, or `/g`, so we write: `s/match/substitution/g`.

## *Capturing* a user's R packages

**Note**: *I am using [regex101](https://regex101.com) for many of my code examples.  It's a very useful too, and all complete regular expressions are linked to a page showing how they work in the context of the examples I provided above.  I have another post on [regular expressions in R](http://www.goring.org/resources/tutorial.html) that may also be of interest.*

### Capturing `library` calls

In general the first few cases above, where `library()` is used to call the package, should be relatively straightforward to match.  Regex doesn't just match complete strings, it allows you to use capture groups, specified elements within the full regex match.  So for example the regex [`^library\((.+)\)`](https://regex101.com/r/5IAcqe/1) will capture (1) any occurrence of `library()` (2) at the beginning of a line (indicated by the `^`), (3) with brackets (we need to escape them using `\(` or `\)`) (4) enclosing some text (`.`) (5) of length one or more (`+`).  By putting the string `.+` in brackets we tell the regular expression engine that this match is a special part of the regular expression, a *capture group*.

In most regex engines, the capture groups can be returned using the notation either `$1` or `\1`.  So we could match `library(`**`ggplot2`**`)` with `^library\((.+)\)` and return `ggplot2`.  You can try this out in the terminal using:

```
echo 'library(ggplot2)' | sed 's/^library[(]\(.*\)[)]/\1/p'
```

### Process the `sed` output

The challenge now is that the capture string still gets a variety of library calls, whether quoted (`library("ggplot2")`), or in lists of packages (`library(ggplot2, cars)`), or quoted lists.  To manage that we need to use bash pipes and a little function called [`tr`](https://ss64.com/bash/tr.html), so we can clean up any extraneous characters and turn the packages into an array that can be used in bash.  Try this:

```bash
echo 'library("ggplot2", neotoma, "dplyr", verbose = FALSE)' | \
  sed -n 's/^library[(]\(.*\)[)]/\1/p' | \
  tr "," "\n" | \
  tr -d "[\"\\']" | \
  sed "s/verbose\s*=\s*\(\(TRUE\)\|\(FALSE\)\)/ /g"
```

The `sed` matching (with the `-n` option) is piped out. With the first `tr` we translate all occurrences of `,` to a carriage return (`\n`).  The second `tr` deletes (`-d`) occurrences of single or double quotes. We have to escape the quotes otherwise bash would think it was the end of the quoted text, and we place quotes in square brackets to say that either type of quote is acceptable.  The last `sed` command is used to remove the option `verbose = FALSE` or `verbose = TRUE` which may or may not be present in the library command. You can see [this line within the context of the final bash script in my GitHub Gist](https://gist.github.com/SimonGoring/fe3b0bf6b4cb9b8f0a2aa6cdce10bb29#file-install_libs-sh-L17).

The code-block above gives a list of packages, separated by a hard return (`\n`).  In a `bash` script we assign the list to a variable; we can see that things are working by writing the bash file and then executing it from the command line:

```bash
#!/bin/bash

library=$(cat R/*.R | sed -n 's/^library[(]\(.*\)[)]/\1/p' | tr "," "\n" | tr -d "[\"\\']" | sed "s/verbose\s*=\s*\(\(TRUE\)\|\(FALSE\)\)/ /g")

echo $library
```

[We can add a second line](https://gist.github.com/SimonGoring/fe3b0bf6b4cb9b8f0a2aa6cdce10bb29#file-install_libs-sh-L18), replacing `library` with `require`, giving us the first set of match requirements.

### Matching package imports in `roxygen2`

In `roxygen2`, and when people are building packages, it's possible to call packages using the statemetn `@import` or `@importFrom`.  Here we would either declare a single package, or a package and then subsequent functions from that package.  For valid `roxygen2` markup this needs to be proceeded with `#'`, so we can look for something like this:

[`^\s*\#+\'\s+\@import\(\?:From\)\?\s\([[:alnum:]]+\)`](https://regex101.com/r/bEuyvd/2)

We begin the line (`^`), possibly with space (`\s*` allows zero or more), followed by the special `#'` character (escaped, and allowing for one or more comment characters: `\#+\'`) with at least one space (`\s+`) and then `@import` which could be followed by `From` (with escaped parentheses followed by a question mark to indicate an optional match: `\@import\(From\)\?`).  The capture group here is defined only as `[[:alnum:]]` a regex class of all alphanumeric values.  This is different than the earlier request where we captured `(.*)` because in the library call we expected to potentially obtain comma separated lists, and we needed to account for the possibility of quoted package names.  This then becomes the [third match in the bash script](https://gist.github.com/SimonGoring/fe3b0bf6b4cb9b8f0a2aa6cdce10bb29#file-install_libs-sh-L19).

### Matching from pipes (`%<%`)

The regular expressions to capture calls within pipes also catches any call where a function is called with its package explicitly, using `package::function()`.  The call requires the use of `perl` rather than the initial `sed` since `perl` allows the use of optional matches, where `sed` does not.

```bash
perl -pe 's/(.*?)([[:alnum:]]+)(::)(.*?)|./ \2/g' | sed '/^\s*$/d')
```

The regular expression ([`(.*?)([[:alnum:]]+)(::)([[:alnum:]]+?)|.`](https://regex101.com/r/HOnP3o/1)) matches any set of alphanumeric text that is followed by `::`, indicating that it is the package calling the function.  The function name, indicated by the second `([[:alnum:]]+?)` indicates a [lazy match](https://docs.microsoft.com/en-us/dotnet/standard/base-types/quantifiers-in-regular-expressions#greedy-and-lazy-quantifiers), which tries to match as few elements as possible.  We follow this with the `.`, so that it gives the lazy match something to stop on (a space, a pipe, whatever).

We have to use `perl` in this case since `sed` does not recognize non-greedy matches, but the options here are the same.  The `perl -e` flag executes the command in the quotes, and, as before, the `-p` flag prints the output.  This winds up matching a lot of empty space, which is unfortunate, but the `sed` match then removes any line that contains only spaces to the end of the line: `\s*$`. This completes [the set of regex calls we need for the `bash` script](https://gist.github.com/SimonGoring/fe3b0bf6b4cb9b8f0a2aa6cdce10bb29#file-install_libs-sh-L20).

## Cleaning an array of packages

Each of these regular expression/perl/sed sequences will return a set of package names.  [In the `bash` file these are aggregated into a single long array by chaning them](https://gist.github.com/SimonGoring/fe3b0bf6b4cb9b8f0a2aa6cdce10bb29#file-install_libs-sh-L17-L20). In some cases the returns from these calls may be separated by only a single space. [Passing the `library` array into `tr` and replacing spaces with hard returns (`\n`)](https://gist.github.com/SimonGoring/fe3b0bf6b4cb9b8f0a2aa6cdce10bb29#file-install_libs-sh-L22) gives an array of libraries that we can sort using unique values, returning the unique set of packages.

So the bash file:

```bash
#!/bin/bash

library=$(cat $(find . -type f \( -name \*.R -o -name \*.Rmd \)) | sed -n 's/^library[(]\(.*\)[)]/\1/p' | tr "," "\n" | tr -d "[\"\\']" | sed "s/verbose\s*=\s*\(\(TRUE\)\|\(FALSE\)\)/ /g")
library+=$(cat $(find . -type f \( -name \*.R -o -name \*.Rmd \)) | sed -n 's/^require[(]\(.*\)[)]/ \1/p' | tr "," "\n" | tr -d "[\"\\']" | sed "s/verbose\s*=\s*\(\(TRUE\)\|\(FALSE\)\)/ /g")
library+=$(cat $(find . -type f \( -name \*.R -o -name \*.Rmd \)) | sed -n 's/^.*\@import\(From\)\?\s\([a-zA-Z]*\)\s.*/ \2/p')
library+=$(cat $(find . -type f \( -name \*.R -o -name \*.Rmd \)) | perl -e 's/(.*?)([[:alnum:]]+)(::)(.*?)|./ \2/g' | sed '/^\s*$/d')

installs=$(tr ' ' '\n' <<< "${library[@]}" | sort -u | tr '\n' ' ')

echo $installs

```

Returns `Bchron dplyr fields ggplot2 gridExtra maps mgcv neotoma plyr purrr purrrlyr raster readr reshape2 rgdal rmarkdown svglite viridis` if you locally clone [a project I am currently working on](https://github.com/Paleon-project/stepps-baconizing).  If the user is not interested in installing the packages the results may look like this:

```bash
 The package ggforce hasn't been installed.
 The package ggmap hasn't been installed.
 The package giphyR hasn't been installed.
 The package gstat hasn't been installed.
 The package hdf5 hasn't been installed.
 The package highlight hasn't been installed.
```

Using the script without installing packages may be a good first step, since it will indicate the extent to which packages are required, and also, the bash script is working with the current set of R scripts.  For this reason, we use an installation flag.

## Installing packages

In the bash script I allow the flag `-i` using [a set of commands at the top of the `bash` file](https://gist.github.com/SimonGoring/fe3b0bf6b4cb9b8f0a2aa6cdce10bb29#file-install_libs-sh-L5-L13):

```bash

rinstall=0

while getopts "i" OPTION
do
	case $OPTION in
		i)
			echo Running installLib with the option to install packages.
			rinstall=1
			;;
	esac
done

```

This uses the bash [`getopts` command](https://ss64.com/bash/getopts.html), and echos to the screen when a user chooses to install the packages.  If they've chosen to install the packages then we need to check each package name against the set of currently installed packages.

### Is the package installed?

First we need the current path for R libraries, which we obtain from the command [`.libPaths()`](https://stat.ethz.ch/R-manual/R-devel/library/base/html/libPaths.html).  Assuming R can be called globally, we can execute `rpath=$(Rscript -e "cat(.libPaths())")` in the `bash` script. This sets an internal bash variable to the array of paths.  For each path element we test whether any of the packages are already installed by looking for the directory using `test -d "$paths/$onePkg"`.  If the package is not present then `install` remains `0`, otherwise it is changed to `1`.

From there we use the equalities to test whether to install the package or not.  If the package isn't installed and the flag has not been set, then simply print to the screen:

```bash
test $install -eq 0 && printf "  The package %s hasn\'t been installed.\n" $onePkg
```

Otherwise, if the flag `-i` has been used, then install the package from the main `cran` repository:

```bash
test $install -eq 0 && test $rinstall -eq 1 && printf "  * Will now install the package.\n" && Rscript -e "install.packages (\"$onePkg\", repos=\"http://cran.r-project.org/\")"
```

## Wrapping it up

So, at this point, we can `git clone`, and copy our bash script (wherever it is) into the cloned directory:

```
git clone git@github.com:SimonGoring/RegularExpressionR.git
cp installLib.sh ./RegularExpressionR/installLib.sh
cd ./RegularExpressionR
bash installRpkg.sh -i
```

and we will have all of our packages installed.  For me, this is a huge time saver.  If you have suggestions, comments, or want to use the script, check it out of my [GitHub gist](https://gist.github.com/SimonGoring/fe3b0bf6b4cb9b8f0a2aa6cdce10bb29).  Feel free to comment or edit anything you need.

### Caveats

The whole script eventually runs through all R files and checks them all, pulling all the packages and then running the `install.packages()` command through `Rscript`.  As mentioned before this will not work on all of the possible options for installing packages, but in most cases, failures will generally either result in trying to install invalid packages (e.g., *TRUE* or *=*), or it will fail to detect a package call.  In addition, this will not install packages that are installed using `devtools::install_github()`, however, it will install `devtools`, and, subsequently, if `install_github()` is called explicitly within the scripts, then the package should be installed.
