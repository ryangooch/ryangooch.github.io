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

I realized upon moving to Fort Collins, though, that beer could be so much more than the mass marketed Coors, Budweiser, and Miller offerings. There were hundreds of styles of it for one thing - beer not being a style itself, but a broad umbrella term encompassing things like Imperial Stouts, Amber Ales, Porters, Bocks, Stouts, IPAs, Sours, and 100 more options ([craftbeer.com](https://www.craftbeer.com/beer-styles) has a nice, visually appealing guide to many styles if you're interested). People making beer were experimenting with so many kinds of flavors, aging regimes, ingredients, and brewing methods. It wasn't just your friend's NASCAR-loving father who enjoyed beer, it was something that was approachable by everyone, allowing almost every drinker a chance to find something unique that they prefer.

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
In order to get the beer data in, I went through the usual thought processes. Is there an open data set somewhere for this? Should I try to scrape the information from [ratebeer.com](https://www.ratebeer.com/) or [beeradvocate.com](https://www.beeradvocate.com/), assuming they were OK with it? Is there are API I can take advantage of?

In the end, I opted for the latter. [BreweryDB](http://www.brewerydb.com/) has both a free and premium API for developers looking to build apps atop beer data. At first I played around with the free version, but decided to go for the Premium option to get access to everything they had. I had some ideas (of which this post is the first) to implement, and down the road I'll need more flexibility than what the free option entails.

I downloaded information on all the beers they had, then coerced the data into a format I liked. Since I focused on the descriptions for this first effort, I simply excluded all entries where there was no description.

### Hopping To Analysis
I relied on my knowledge from previous projects for the natural language processing aspects, such as [What's Cooking](https://www.kaggle.com/c/whats-cooking), as well as some resources I found online, including [ultravioletanalytics.com](http://www.ultravioletanalytics.com/2016/11/18/tf-idf-basics-with-pandas-scikit-learn/) and [untrod.com](http://blog.untrod.com/2016/06/simple-similar-products-recommendation-engine-in-python.html). My text processing approach was to

  1) Use the fabulous [Natural Language Toolkit](http://www.nltk.org/) to find the stems (or roots) of the words in the descriptions;
  2) Employ [scikit-learn's CountVectorizer](http://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html) to count up the number of occurrences of keywords, and;
  3) Use [term-frequency inverse-document-frequency](http://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfTransformer.html#sklearn.feature_extraction.text.TfidfTransformer) to assign weights to each keyword
  
Following these steps, I could use cosine similarity to compare a particular set of keywords for a particular beer to the dataset and grab the top most similar beers to it. I decided to compute this "on the fly", since for one thing, computing it for each entry at once resulted in MemoryErrors, and while I found a few methods around this, I decided it wasn't exactly worth it, when computing one at a time is extremely fast and all I'll need (for now).

[Cosine similarity](https://en.wikipedia.org/wiki/Cosine_similarity) itself is a measure of the orientation of one vector with another. The cosine angle between the two vectors would give similarity result, and this value ranges from 0 to 1, where 1 is perfect similarity and 0 is totally different. Magnitude is irrelevant here, which is kind of nice for our case; we don't really care about magnitude, just the relative distributions of keyword weights. 

$$
\begin{equation}
\texttt{similarity} = \cos{\theta}= \frac{ \textbf{A} \dot \textbf{B}} { || \textbf{A} || || \textbf{B} || } \\
\end{equation}
$$

## So How Does It Taste?
I can choose any beer in the database, but decided to use Fort Collins beers as my ~~taste~~ test case. First up is Odell's 90 Shilling, their flagship beer, an amber ale. The resulting most similar craft beers are:

| Name                  | Similarity |
| --------------------- | ---------- |
| 90 Shilling           | 1.00       |
| Harvester Pumpkin Ale | 0.41       |
| Scotch Ale            | 0.39       |
| Leisure Winter Ale    | 0.38       |
| Etna                  | 0.38       |

I omitted the descriptions and breweries here, but I will say this is a pretty reasonable top 5. Four of these are amber ales, and all 5 are described as "medium-bodied". The Pumpkin Ale is a bit of an outlier, but if you like 90 Shilling, then you would have a good Autumn beer to try out next time it is in season. The Scotch Ale is a bit different than the Amber Ale, but it was included since in mentioned it had a rich flavor and was medium-bodied. Etna and the Winter Ale both boast a balance with hop flavor. 

In fact, I'm pretty pleased with the result. I didn't want too fine a comb for the model, since this is in spirit a way to find and try new beers. If we simply tried other amber ales, then we would find good options, but these results include some new styles to try as well.

For comparison's sake, take a look at 90 Shilling's Description in the data set...

> We introduced 90 Shilling, our flagship beer, at our opening party in 1989. For a while, we’d been wondering what would happen if we lightened up the traditional Scottish ale? The result is an irresistibly smooth and delicious medium-bodied amber ale. The name 90 Shilling comes from the Scottish method of taxing beer. Only the highest quality beers were taxed 90 Shillings. A shilling was a British coin used from 1549 to 1982. We think you’ll find this original ale brilliantly refreshing, and worth every shilling.

... and the Scotch Ale's:

> This delicious medium-bodied Ale is brewed in the traditional Scottish Ale style, using no less than six quality malts. The result is a deep ruby coloured ale with a rich, robust flavour and notes of caramel. Best enjoyed between 6-8ºC.

Both look pretty tasty to me, and you can see where the similarity exists in these two differing styles.

For those bitter fans out there, upset I haven't included you, take a look at the beers most similar to Odell's Myrcenary IPA:

| Name                  | Similarity |
| --------------------- | ---------- |
| Myrcenary             | 1.00       |
| El Dorado IPA         | 0.69       |
| Neanderthal           | 0.66       |
| Flip Your Whig        | 0.66       |
| La Salamandre         | 0.66       |

Here the algorithm exhibits some weakness, keying in on the term 'Double IPA'. In fact, Neanderthal and Flip Your Whig contain only that text as their entire description. That doesn't mean they're bad things to try, just that the algorithm is recommending based exclusively on that. Going a little further down, however, we get a nice description for Fully Adrift Double IPA (similarity = 0.66),

> 2014 Alpha King Competition runner up and 2015 L.A. International Beer Competition Silver Medal winner for Double IPA. Fully Adrift is a massively hopped, sticky, dank, tropical, resinous and magical Double IPA brewed from a secret combination of hops added at over 8lbs per bbl! Key Descriptors – Hop forward, Dank and Sticky, Tropical

compared with Myrcenary,

> Named for Myrcene, a component of essential oils in the hop flower, Myrcenary Double IPA is our tribute to those who revere the illustrious hop, and their unyielding exploit to craft hop forward beers. Brewed with a blend of hops containing the highest levels of Myrcene, this double IPA prevails with a tropical fruit-like flavor, a pungent floral aroma, and a clean getaway.

These look like two good beers to try for those hopped-up fans out there.

Finally, let's check out a sour, for all you sour lovers out there. I'll focus on Le Terroir from New Belgium.

| Name                    | Similarity |
| ----------------------- | ---------- |
| Le Terroir              | 1.00       |
| Old Guardian Dry Hopped | 0.40       |
| Sideshow                | 0.38       |
| Black is the New Wit    | 0.38       |
| Barrel Aged Elysium     | 0.35       |

This is an interesting collection featuring a few sours, a wit, and an imperial stout. The keywords of importance seemed to have been barrel-aged and sour-ale. It's not maybe as interesting a result as it could be, but I don't see these as bad suggestions. I think the sour collection in the database in general needs some help, as either many sours are excluded in the dataset, or fell victim to my description filter.

## Final Thoughts
This was an interesting experiment, and I've wanted to play with some beer data for a while. Some features to add to this would be brewery and location data, and incorporating things like ABV and IBU into the recommendation system, as well as categorical variables for style information. Maybe a stacked classification where descriptions are used first, then a classifier to add in the quantitative characteristics of each description? I could impute values to deal with missing data based on styles fairly easily. And maybe down the road, I can build a really interesting recommendation system with ratebeer or Untapped data!
