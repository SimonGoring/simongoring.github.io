---
layout: page
res_class: Development
title: Building a Python Twitter-bot
concept: Building outreach tools & learning programming languages from concepts.
---

# Explorations in outreach – Creating a Twitter bot for the Neotoma Paleoecological Database.

If you’ve ever been in doubt about whether you chose the "right" programming language I want to lay those concerns to rest.

For many researchers, particularly in Biology or the Earth Sciences, there is often a question about whic programming language is the most useful, R, Python, Matlab . . .  This choice is especially daunting if you’re first approaching scientific programming in grad school.  You already feel like you don’t know anything, and, ultimately, you're likely to wind up learning whatever you’re taught, or whatever your advisor is using and you wonder. . . Is the grass greener over in Python-land? Those figures look nice, if only I had learned R. . . Why did I learn on an expensive closed platform?

I am here to say “Don’t worry about it”, and I'll emphasize with an example centered around academic outreach:

<img src="/images/Packrat.png" style="float:left; margin-right:10px">
The [Neotoma Paleoecological Database](http://neotomadb.org/) has had an issue for several years.  A large number of datasets have been submitted, but few people could upload datasets directly to the database.  Neotoma is a living database.  Not only do new datasets get added, but, as new information becomes available (for example, new taxonomic designations for certain species) datasets get updated.  This means that database curation is time intensive, leading to a gap between data submission and data publication.  To make up for this there has been a data “Holding Tank” where individual records have been available, but this wasn’t the best solution.

Fast forward to about a year ago. Eric Grimm update the software package [Tilia](http://tiliait.com) to provide greater access to the database to a set of domain experts who would act as data stewards.  Each [data type](http://neotomadb.org/data) has one or a few stewards who can vet and upload datasets directly to the database using *Tilia*. This increased the speed at which datasets are uploaded -- over the last month there have been more than 200 new datasets entered -- but it’s still hard to get a sense of this as an outsider.

## A new venue for outreach - Twitter

One solution involves [Twitter](http://twitter.com). Academics use of Twitter increased rapidly, although it's likely plateaued.  Wired has posted a list of [27 Twitter feeds for Science](http://www.wired.com/2015/08/the-new-cultural-literacy-science-feeds/), Science published a somewhat contentious list of [50 science stars to follow](http://www.sciencemag.org/news/2014/09/top-50-science-stars-twitter), and [I’m on Twitter](http://twitter.com/sjGoring), so obviously all the cool kids are there. 

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Just letting you know I&#39;ve got an account on Twitter.</p>&mdash; Simon Goring (@sjGoring) <a href="https://twitter.com/sjGoring/status/767948864737325056">August 23, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## Planning it out:

All this Twitter action led me to think it could be a good platform for publicizing new data uploads to Neotoma. I just needed to learn how. Theoretically, the process was fairly straightforward:

1. Figure out what the most recently posted Neotoma datasets are:
	* Made easier with the Neotoma API, which has a specific method for returning recently uploaded datasets: `http://ceiwin10.cei.psu.edu/NDB/RecentUploads?months=1`
	* You’ll notice (if you click) that the link returns data in a weird format.  This format is called [JSON](http://www.json.org/) and it has been seen by many as the successor to XML (as the [JSON website explains](http://www.json.org/xml.html) - bias should be obvious).
2. Check it against two files, (1) a file of everything that’s been tweeted already, and (2) a file with everything that needs to be tweeted (since we’re not going to tweet everything at once)
3. Append the new records to the queue of sites to tweet.
4. Tweet.

So that’s it (generally).  I’ve been working in R for a while now, so I have a general sense of how these things coud be put together in R, but the mechanics of the app translate to other languages as well. The hardest thing about programming (in my opinion) is figuring out the program flow. Everything else is just window dressing. Once you get more established with a programming language you’ll learn the subtleties of the language, but for hack-y programming, you should be able to get the hang of it regardless of your language background.

As evidence, [Neotomabot](https://github.com/SimonGoring/NeotomaBot/). The code’s all there, I spent a day figuring out how to program it in Python. But to help myself out, I planned it all first using long-hand notes, and then hacked it out using Google, StackOverflow and the Python manual. Regardless, it’s the flow control that’s the key. With my experience in R I’ve learned how “for” loops work, I know about “while” loops, I know `try-catch` methods exist and I know I need to read JSON files and push out to Twitter. Given that, I can map out a program and then write the code, and that gives us Neotomabot:

## Coding it out (sort of):

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Neotoma welcomes Lac Brule, a pollen dataset by K. Lafontaine-Boyer <a href="https://t.co/atbdCdFDbi">https://t.co/atbdCdFDbi</a></p>&mdash; Neotoma Database (@neotomadb) <a href="https://twitter.com/neotomadb/status/767829739721465856">August 22, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

All the code is available on the GitHub repository [here](https://github.com/SimonGoring/neotomabot), except for the OAuth handles, but you can learn more about that aspect from [this tutorial](http://www.dototot.com/how-to-write-a-twitter-bot-with-python-and-tweepy/) that I found useful for getting started.  There is also a `twittR` package for R, with several tutorials ([here](http://davetang.org/muse/2013/04/06/using-the-r_twitter-package/), and [here](http://www.r-bloggers.com/getting-started-with-twitter-in-r/)) if you choose to do this all in R.

So that’s it.  You don’t need to worry about picking the wrong language. Learning the basics of any language, and how to map out the solution to a problem is the key.  Focus on these and you should be able to shift when needed.

As proof, a fantastic undergrad in the [Williams Lab](http://www.geography.wisc.edu/faculty/williams/lab/), Ashtin Massie, went over and above my little hack to produce the greatest weather-bot ever:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">It&#39;s the MOSt important time of the day! Here&#39;s what the models are forecasting for the next few days. <a href="https://t.co/GO0XRXjMjQ">pic.twitter.com/GO0XRXjMjQ</a></p>&mdash; Madison Weather Bot (@MSNWeatherBot) <a href="https://twitter.com/MSNWeatherBot/status/762762950004269056">August 8, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Awesome, right?!