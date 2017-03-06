---
layout: post
title: Building a Craft Beer Recommendation System with Python, Scikit-Learn, and Pandas
date: 2017-03-05 17:00:00
tags: craft beer, brewerydb, python, pandas, scikit-learn, content-based, recommendation, odell
comments: true
---

## Project Summary

I built a simple, craft beer recommendation system using tools from natural language processing with python, pandas, and scikit-learn. To use it, I can input the name of a craft beer that is present in the database, then compute the cosine similarity between a bag-of-words representation of each beer's description with all others. The code (gist) is found [here](https://gist.github.com/ryangooch/7c620807bf0266ba8569113a378b77eb), though the dataset will have to be compiled by the user!

## Craft Beer Proliferation

Prior to moving to Fort Collins, Colorado, I didn't really "get" beer. As I reached college in Kentucky, I was more likely than not to be drinking a bourbon or a scotch than beer. Sure, I'd swill a Bud Light, but it wasn't the most flavorful experience, didn't pack a punch, and generally was far less interesting. I thought then that beer was pretty much what it was to the largest suppliers; lowest common denominator in appeal and quality.

I realized upon moving to Fort Collins, though, that beer could be so much more than the mass marketed Coors, Budweiser, and Miller offerings. There were hundreds of styles of it for one thing - beer not being a style, but a broad umbrella term encompassing things like Imperial Stouts, Amber Ales, Porters, Bocks, Stouts, IPAs, and 100 more options ([craftbeer.com](https://www.craftbeer.com/beer-styles) has a nice, visually appealing guide to many styles if you're interested). People making beer were experimenting with so many kinds of flavors, aging regimes, ingredients, and brewing methods. It wasn't just your friend's NASCAR-loving father who enjoyed beer, it was something that was approachable by everyone, allowing almost every drinker a chance to find something unique that they prefer.

The choices were overwhelming for me at first. Fort Collins has a lot of breweries for a small-ish town, upwards of 20 with new ones popping up every year. And then, once you've chosen a taproom, you're confronted with a vast array of choices from that brewery alone, drawing on many of the styles mentioned above. How does a person know what to pick? Assuming they try something they like, what informs the next decision on beer to try?

![Odell Beer Patio](http://www.visitftcollins.com/wp-content/uploads/2015/04/Odell-beer-patio.jpg "Odell Beer Patio")
Figure 1. Odell Beer Patio, dog- and kid-friendly tap areas like this pepper the Fort Collinsian landscape. Courtesy of visitftcollins.com

My approach was like many others.

  1) Meet up with friends.
  2) Pick out a brewery.
  3) Bike to said brewery on one of the many gorgerous Fort Collins afternoons.
  4) Try beers at pseudo-random until I found a few styles I liked.
  5) Try some crazy beers (spicy chocolate stouts ftw)
  6) Go for a tour
  7) Walk my bike home
  
This is a fun strategy, and is a great way to spend a Saturday in the Front Range. But it seems inefficient, especially in this age of computing power, analytics, and Big Data, right? Surely, there is a beer database with which we can intelligently find beers we like using some form of a recommendation system? It turns out, we can!

## Drunken Machine Learning

I had this great idea for a "Pandora of beer" a while back, where you could input a few beers you like and dislike, then have beers recommended to you. This is an entirely reasonable task, but not something I've ever devoted much time to realizing. Maybe one day. For now, though, I figured an interesting task might be to simply query a database for similar beers to a given input as a way to inform someone as to other beers they might like. The basic process was as follows:

  1) Download information on a bunch of craft beers
  2) Massage the data into a format that was useful
  3) Use keywords from the descriptions of the beers to find pairwise similarities
  4) Store the top 20 similar beers for each
  
### Ingesting Tens of Thousands of Beers

In order to get the beer data in, I went through the usual thought processes. Is there an open dataset somewhere for this? Should I try to scrape the information from [ratebeer.com](https://www.ratebeer.com/) or [beeradvocate.com](https://www.beeradvocate.com/), assuming they were OK with it? Is there are API I can take advantage of?

In the end, I opted for the latter. [BreweryDB](http://www.brewerydb.com/) has a free and premium API for developers looking to build apps atop beer data. At first I played around with the free version, but decided to go for the Premium option to get access to everything they had. I had some ideas (of which this post is the first) to implement, and  down the road I'll need more flexibility that what the free option entails.

I downloaded information on all the beers they had, then coerced the data into a format I liked. Since I focused on the descriptions for this first effort, I simply excluded all entries where there was no description.

### Hopping To Analysis

I relied on my knowledge from previous projects for the natural language processing aspects, such as [What's Cooking](https://www.kaggle.com/c/whats-cooking), as well as some resources I found online, including [ultravioletanalytics.com](http://www.ultravioletanalytics.com/2016/11/18/tf-idf-basics-with-pandas-scikit-learn/) and [untrod.com](http://blog.untrod.com/2016/06/simple-similar-products-recommendation-engine-in-python.html). My text processing approach was to

  1) Use the fabulous [Natural Language Toolkit](http://www.nltk.org/) to find the stems (or roots) of the words in the descriptions;
  2) Employ [scikit-learn's CountVectorizer](http://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html) to count up the 
