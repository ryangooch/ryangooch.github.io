---
layout: post
title: Calculating Massey and Colley Ratings on NCAA Basketball Data
date: 2017-2-12 18:00:00
tags: python, pandas, ncaa, ncaa tournament, massey, colley
comments: true
---

## Project Summary 
Using data from [Kaggle's Machine Learning Mania]() contest, I calculated Massey and Colley ratings for each team in one season using python, pandas, and numpy. You can see my script with exploratory analysis, some pandas tricks, and the rating calculations [here](https://gist.github.com/ryangooch/da9bc9be3d2a5a204b9db2cfc369a128).

## Problem Statement
In my previous posts I have detailed the process by which I have seeded brackets in the past. The basic process is to download the ranking data from [masseyratings.com](http://www.masseyratings.com/) for all computer and human polls, then aggregate the data with simple formulas to produce a final ranking that could be used to seed a bracket.

My goals in the beginning were to come up with a very simple process that could be implemented to seed an NCAA tournament bracket by hand (technically, Excel) in about an hour's time, or in an automated fashion in seconds. I wanted to refute the Selection Committee's contention that their time constraints, along with the rules for the tournament, led to an extremely difficult weekend of seeding, which might lead to poorly seeded brackets. For this purpose, the above model works. However, there have always been a few issues with this model, including:

* Aggregating Ranks instead of Ratings
* Redundant information biasing final results

There are probably a few more, but these are the two biggest ones I've wanted to tackle for a while. I'll speak mostly to the first bullet in this post.

### Aggregating Ranks Instead of Ratings
It might be good to explain the difference between these two related concepts before proceeding. A rank is an ordered list of integers denoting a comparative description of the items in said list. It is a permutation of integers 1 through n, where n is the number of items in the list. A rating is a numerical score for each item in a list, that, when sorted, creates a ranked list.

This is better illustrated by a table. Take a look at the following table, which reports 5 teams from the 2010-11 NCAA basketball season, with associated ranks and ratings. Don't worry about what Massey and Colley mean for now! We will come back around to it shortly.

|	Team |	Massey |	Colley |	Massey_rank |	Colley_rank |
| --- | --- | --- | --- | --- |
|Ohio St	|26.0265	|1.07358	|1|2|
|Duke	|25.6505	|1.02081	|2	|4|
|Kansas	|25.5452	|1.07924	|3	|1|
|Texas	|22.0774	|0.93496	|4	|12|
|Pittsburgh	|21.9288|	0.996594	|5|	5|
|Washington|	21.3346|	0.813653|	6|	33|
|Kentucky|	20.8808	|0.921453	|7	|14|
|Purdue	|20.6198	|0.931967|	8	|13|
|Louisville	|19.8196|	0.915334	|9	|15|
|Syracuse	|19.7607|	0.95558	|10	|9|
Table 1. Selection of teams with ratings and rankings from 2011 NCAA basketball season

The values in Table 1 were computed as part of the analysis described in this post, and represent the top 10 ranked teams according to the Massey Rating for the entirety of the 2010-11 season. Notice that in the 'Massey" column with have a group of scores, while "Massey Rank" contains ordinal values, with ranks derived directly from the comparative scores of the Massey rating. The same goes for Colley.

So, what's the big deal? The ranks preserve information about the ratings, and all we REALLY want to know is if team A is better than team B, right?

Well, I'm not so sure. One problem here is, how much *better* is team A than team B? Ranks don't preserve this information. In fact, I believe that they can *distort* that information. What I mean by that is, the team ranked number 1 by Massey, Ohio St, might appear to be *twice as good* as Duke, if you look at rank alone. We can look at their *percent difference* to see this;

(note to self; see http://gastonsanchez.com/visually-enforced/opinion/2014/02/16/Mathjax-with-jekyll/ for equation support)
percent difference = abs((rank Team A - rank team B)) / (rank team A) * 100 %

In this case, Ohio State might appear to be 100 % better than Duke, or twice as good! This is unlikely to be true. The distortion aspect comes into play when we look at Louisville vs Syracuse, which makes Louisville appear to be 11% better than Syracuse. By the time you reach teams ranked around the 40 mark (normally the cut-off for at-large teams in the tournament), the difference is even smaller. Again, it's a possibility that this is the distribution of a team's objective ability, but not the only possibility, so it represents a problem.

Another issue is that aggregating ordinal values (ranks) by taking an average coerces them into non-ordinal (but still real) numbers. In other words, it takes ranks and puts an aggregate of ranks into a rating. Which feels statistically dirty to do. In other (other) words, we compare apples to oranges by forcing everything to look like an apple when they're really oranges. Why not compare apples to apples (ratings) and convert to oranges (ranks) at the end? OK that's a pretty terrible though example but it made sense to me earlier today so maybe it will help you. If not, read on, it gets better I promise.

### Aggregating Ratings, Not Ranks
So, we want to improve the bracket selection method by aggregating ranks, not ratings. That makes sense. But how do we do that? Well, there are smart ways to do this and we will cover one in the near future. For now, though, we can compute these ratings, but we still can't really compare them. Both Massey and Colley have differing ranges (min to max values), and variances (how much they deviate from whatever their mean is). Luckily, we have a tool at our disposal here: standardization. We can put each array into terms of each's mean value and its variance. Then, we can average the values together for a simple first-pass aggregation.

So what do the standardized ratings look like? Take a look at Table 2.

|	Team |	Massey |	Colley |	Massey_rank |	Colley_rank | Massey_scaled | Colley_scaled |
| --- | --- | --- | --- | --- | --- | --- |
|Ohio St	|26.0265	|1.07358	|1|2| 2.572 | 2.482 |
|Duke	|25.6505	|1.02081	|2	|4| 2.535 | 2.254 |
|Kansas	|25.5452	|1.07924	|3	|1| 2.524 | 2.507 |
|Texas	|22.0774	|0.93496	|4	|12| 2.182 | 1.882 |
|Pittsburgh	|21.9288|	0.996594	|5|	5| 2.167 | 2.149 |
|Washington|	21.3346|	0.813653|	6|	33| 2.108 | 1.357 |
|Kentucky|	20.8808	|0.921453	|7	|14| 2.063 | 1.824 |
|Purdue	|20.6198	|0.931967|	8	|13| 2.038 | 1.869 |
|Louisville	|19.8196|	0.915334	|9	|15| 1.959 | 1.798 |
|Syracuse	|19.7607|	0.95558	|10	|9| 1.953 | 1.972 |
Table 2. Same as Table 1, with scaled Massey and Colley ratings

From here, it is a little more defensible to do things like take averages and perform other numerical methods of aggregation, which we can do now trivially. However, we're putting the cart before the horse; how do we actually manipulate the data and calculate these Massey and Colley Ratings? What does the math look like?

## Analysis
Unless otherwise noted, I'm following the treatment in the excellent [Who's #1? The Science of Rating and Ranking](http://www.goodreads.com/book/show/13530569-who-s-1-the-science-of-rating-and-ranking) by Amy Langville and Carl Meyer. It should also be noted that [the script I've posted](https://gist.github.com/ryangooch/da9bc9be3d2a5a204b9db2cfc369a128) has some additional analysis and data cleaning to get it into the form I wanted, including some decent date/time manipulation and indexing. Some of the analysis there was based on a [Kaggle Kernel by ralston3](https://www.kaggle.com/ralston3/march-machine-learning-mania-2016/ncaam-exploratory-analysis), so I thank them for their work and allowing some random analyst on the internet to use it and learn a thing or two from it!

### Massey Ratings
Massey ratings are based on score differential. The least squares method assumes that we can predict score differential directly by finding the difference in ratings between any two teams. I won't go through the entire mathematical underpinning for this, as the authors of the book did a great job and I'm just summarizing, but the crucial insight is that we can use a simple system of linear equations to compute this ratings. The general form of the equation is

$$
\begin{equation}
\textbf{M}\textbf{r} = \textbf{p}
\end{equation}
$$

Here, \$$ \textbf{M} $$ is the Massey matrix of size n by n, where n is the number of teams in the league, \$$ \textbf{p} $$ is an n by 1 vector containing the sum of the point differentials for all the games each team has played, and \$$ \textbf{r} $$ is the ratings vector, also n by 1, that we are trying to solve and obtain. The Massey matrix itself contains along the diagonal all games each team has played, with off diagonal elements recording the negation of the number of games any two teams have played. Since there are around 350 teams in D1 basketball, and no team plays more than 31 games in a regular season, most of the off-diagonal elements in the Massey matrix are 0, meaning the matrix itself is sparse. Due to the properties of the matrix, however, we cannot simply invert it to solve the linear systeam of equations without replacing one line in the matrix with 1 in every element, and the corresponding point differential value with 0. This allows a unique solution to the linear equation above, and we can easily solve for the ratings vector.

### Colley Ratings
Wesley Colley set out to find an unbiased estimate of a team's objective value by examing winning percentage. His system, like Massey's, is used in the BCS computer ranking portion for determining bowl berths. The basic idea is that we can use an alteration to the winning percentage formula,

$$
\begin{equation}
r_i = \frac^{w_i}_{t_i}
\end{equation}
$$

test
