---
title: Style your R
concept: A broader vignette of the functionality in the R neotoma package.
layout: page
---

[Advanced R](http://adv-r.had.co.nz/) by Hadley Wickham is a great resource for people working with R.  It provides a good overview and a much deeper dive into some components of functionality within the R programming language.  This short article goes into some of the major concepts in the [Style section](http://adv-r.had.co.nz/Style.html) and builds on them, along with my own suggestions.

# File Naming

Hadley makes the excellent suggestion that file names should be clear and meaningful.  I want to back up though, because I think proper directory structure is the place to start here.  I make the same point over and over, when you start a project it's important to think about how your project is going to look.  A folder with ten `R` files is going to look messy.  Where are your input data files going?  Where are the output files going?  Jenny Bryan has a great link on her website and there is an excellent review from [Software Carpentry](http://software-carpentry.org/lessons/) for [Good Enough Practices](https://swcarpentry.github.io/good-enough-practices-in-scientific-computing/).

Basically, I like to see a repository that looks like this:

```
.
|\- data
|    \- input   [directory]
|    \- output  [directory]
|\- R           [directory]
|\- figures     [directory]
|-- main-R-file.R (or Rmd)
|-- OTHER (github/etc) FILES

```

This then eliminates the need for Hadley's second recommendation about file naming, that sequential files be numbered, *e.g.*, `01_load-data.R`, `02_run-analysis.R`, `03_make-figures.R`.

If you have a nice clean directory structure you can create a "control" file that does the following:

```R
# Parent file, this file calls the other files in order:

source("R/load-data.R")
source("R/run-analysis.R")
source("R/make-figures.R")

```

The advantage here is in simplicity.  If you were running each of those files in order you'd basically be doing the same thing anyway, so why not just formalize it.  That way your directory structure and file names are also cleaner, and you can run the whole thing from the command line easily with one command:

```bash
R < main-R-file.R
```

Now R will run each of those `source()` calls, and you'll have fewer files in each directory.

# Object Names

I don't have many suggestions here, I think that the general ideas here are excellent (obviously).  THe only thing I'd suggest adding here is to try to limit the number of global variables in your namespace.

In some cases the use of general variable names (like `x`, or `i` or `counter`) is unavoidable, and also advantageous.  The problem comes when you have multiple `R` files used in your analysis, it becomes possible to start coding in such a way that you might expect `x` to behave one way in one place, but have made an upstream change where you use `x` in a new and unique way, that changes its behaviour later:

```R
# I want to generate some random numbers:
x <- runif(1000)
y <- runif(1000)

plot(x, y)

# Some extra code
# . . .
# I now expect to use those random numbers to generate noise in a dataset

some_prediction <- predicted_values + x

```

But then a colleague adds a chunk of code to plot some spatial data:

```R
zebra_data <- read.csv("data/zebra-crossings.csv")
x <- zebra_data$long
y <- zebra_data$lat
counts <- zebra_data$counts
png("figures/zebra-crossings.png")
plot(x, y, col = counts)
dev.off()
```

If this code is stuck into a seperate file, sourced where the ellipses are in the upper code block I might never know why my values for `some_prediction` are suddenly so strange.  This is one of the problems of dealing with complex projects, and I've run into it a number of times.

One solution is to wrap everything into functions when you `source()` code.  Pass in only the most important details, pass out only the most critical outputs.  Everything then only lives in the local environment of the function, an is extinguished once the function runs:

```R
plot_zebras <- function(){
  zebra_data <- read.csv("data/zebra-crossings.csv")
  x <- zebra_data$long
  y <- zebra_data$lat
  counts <- zebra_data$counts
  png("figures/zebra-crossings.png")
  plot(x, y, col = counts)
  dev.off()
}

```

In this case, the changes to `x` and `y` happen internally within the function.  You could generalize this function if you wanted, to add parameters that would change the output and input file names, but you don't need to.  Now, if you wrote:

```R
# I want to generate some random numbers:
x <- runif(1000)
y <- runif(1000)

plot(x, y)

# Some extra code

source("plot_zebras.R")
plot_zebras()

# I now expect to use those random numbers to generate noise in a dataset

some_prediction <- predicted_values + x

```
you could expect that you code would run the way you expect, without unexpected changes to `x` and `y`.  I might still question your insistence on using these variables this way, but at least you're not having unexpected behaviour.

# Styling Code

I have two suggestions, one is [the `lintr` package](https://cran.r-project.org/web/packages/lintr/index.html) and the other is recommended in the Style Chapter, [the `formatR` package](https://cran.r-project.org/web/packages/formatR/index.html).

I really like `lintr`.  It automatically integrates into your RStudio edit panel, so you can see mistakes right away:

<figure>
	<img src="/images/lintr_image.png" alt="RStudio R code with lintr icons" align="middle">
	<figcaption>A sample of some poorly formatted R code in RStudio with <i>lintr</i> icons in the sidebar.</figcaption>
</figure>

There are other code profiling tools you could use, but I think that's a deeper discussion, best suited for somewhere else.

# Comments

Comments are good, comments are important, but too many comments are too much.  It's important to remember that good code should be self descriptive.  This involves careful naming of files, functions and variables.  For example, if we re-wrote our zebra code-block, we might require lots of comments to have it make sense to the next person who reads it:

```R
z_dat <- read.csv("data/z-cross.csv")
acrs <- z_dat$ln
updwn <- z_dat$lt
cnts <- z_dat$num
png("figures/z-cross.png")
plot(acrs, updwn, col = cnts)
dev.off()
```

I mean, I guess I see what you're doing, but it could get very complicated.  Especially if all your code looks like this!

So we could add comments:

```R
# Read in Zebra crossing data:
z_dat <- read.csv("data/z-cross.csv")

# Get the latitudes & longitudes:
acrs <- z_dat$ln
updwn <- z_dat$lt

# These are the counts
cnts <- z_dat$num

# Now get it to plot:
png("figures/z-cross.png")
plot(acrs, updwn, col = cnts)
dev.off()
```

Which is fine, but if we just had it the way it was (in the function), the meaning and utility would be clear.  This is also where I depart a bit from Hadley.  I don't like the recommendation of using:

```R
# -----------------------------------------------
```

code blocks.  I think it makes code look a bit ugly.  I'd rather see those sections be blocked into unique files to make navigation easier.
