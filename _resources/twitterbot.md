---
layout: page
title: Building a Python Twitter-bot
concept: Building outreach tools & learning programming languages from concepts.
---

# Explorations in outreach – Creating a Twitter bot for the Neotoma Paleoecological Database.

If you’ve ever been in doubt about whether you chose the "right" programming language I want to lay those concerns to rest.

For many researchers, particularly in Biology or the Earth Sciences, there is often a question about whic programming language is the most useful, R, Python, Matlab . . .  This choice is especially daunting if you’re first approaching scientific programming in grad school.  You already don’t know anything about anything, and, ultimately, you likely wind up learning whatever you’re taught, or whatever your advisor is using and you wonder. . . Is the grass greener over in Python-land? Those figures look nice, if only I had learned R. . . Why did I learn on an expensive closed platform?

I am here to say “Don’t worry about it”, and I'll emphasize with an example centered around academic outreach:

<img src="/images/Packrat.png" style="float:left; margin-right:10px">
The [Neotoma Paleoecological Database](http://neotomadb.org/) has had an issue for several years now.  We have had a large number of datasets submitted, but very few people could actively upload datasets to the database.  Neotoma is a living database -- not only do new datasets get added, but, as new information becomes available (for example, new taxonomic designations for certain species) datasets get updated.  This means that maintaining the database is very time intensive and there has traditionally been a gap between data submission and data publication.  To make up for this there has been a data “Holding Tank” where individual records have been available, but this wasn’t the best solution.

Fast forward to about a year ago. Eric Grimm update the software package [Tilia](http://tiliait.com) to provide greater access to the database to a set of domain experts who would act as data stewards.  Each data type (including insets, pollen, mammal fossils, XRF, ostracodes, lake chemistry) has one or a few stewards who can vet and upload datasets directly to the database using *Tilia*. This has increased the speed at which datasets have entered Netoma -- over the last month there have been more than 200 new datasets entered -- but it’s still hard to get a sense of this as an outsider since people don’t regularly check the database unless they need data from it.

One solution involves Twitter. Academics have taken to Twitter like academics on a grant.  Buzzfeed has posted a list of 25 twitter feeds for nerds, Science published a somewhat contentious list of scientists to follow, and I’m on twitter, so obviously all the cool kids are there. This led me to think that Twitter could be a good platform for publicizing new data uploads to Neotoma.  Now I just needed to learn how.

The process is fairly straightforward:

<ol>
  <li> Figure out what the most recently posted Neotoma datasets are:</li>

  <ul><li>This is made easier with the Neotoma API, which has a specific method for returning datasets: `http://ceiwin10.cei.psu.edu/NDB/RecentUploads?months=1`</li>
  <li>You’ll notice (if you click) that the link returns data in a weird format.  This format is called JSON and it has been seen by many as the successor to XML (see here for more details).</li></ul>

  <li>Check it against two files, (1) a file of everything that’s been tweeted already, and (2) a file with everything that needs to be tweeted (since we’re not going to tweet everything at once)</li>

  <li>Append the new records to the queue of sites to tweet.</li>

  <li>Tweet.</li>
 </ol>

So that’s it (generally).  I’ve been working in R for a while now, so I have a general sense of how these things coud be put together in R, but these same mechanics translate to other languages as well. The hardest thing about programming (in my opinion) is figuring out the program flow. Everything else is just window dressing. Once you get more established with a programming language you’ll learn the subtleties of the language, but for hack-y programming, you should be able to get the hang of it regardless of your language background.

As evidence, [Neotomabot](https://github.com/SimonGoring/NeotomaBot/). The code’s all there, I spent a day figuring out how to program it in Python. But to help myself out I planned it all first using long-hand notes, and then hacked it out using Google, StackOverflow and the Python manual.  Regardless, it’s the flow control that’s the key. With my experience in R I’ve learned how “for” loops work, I know about “while” loops, I know `try-catch` methods exist and I know I need to read JSON files and push out to Twitter. Given that, I can map out a program and then write the code, and that gives us Neotomabot:

 Follow
 Neotoma Database ‎@neotomadb
Neotoma welcomes another North American Pollen Database dataset: Nutella Lake from M. Rohr http://apps.neotomadb.org/Explorer/?datasetid=15809 …
1:19 AM - 5 May 2015
  Retweets   likes

All the code is available on the GitHub repository [here](https://github.com/SimonGoring/neotomabot), except for the OAuth handles, but you can learn more about that aspect from this tutorial: [How to Write a Twitter Bot](http://www.dototot.com/how-to-write-a-twitter-bot-with-python-and-tweepy/). I found it very useful for getting started.  There is also a `twittR` package for R, there are several good tutorials for the package available ([here](http://davetang.org/muse/2013/04/06/using-the-r_twitter-package/), and [here](http://www.r-bloggers.com/getting-started-with-twitter-in-r/)).

So that’s it.  You don’t need to worry about picking the wrong language. Learning the basics of any language, and how to map out the solution to a problem is the key.  Focus on these and you should be able to shift when needed.