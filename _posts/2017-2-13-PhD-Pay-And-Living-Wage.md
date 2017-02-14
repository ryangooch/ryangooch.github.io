---
layout: post
title: Graduate Research Assistant Pay and Living Wage in the US
date: 2017-2-13 21:00:00
tags: gra, graduate research assistant, salary, living wage
comments: true
---

## Project Summary
I compiled a dataset of graduate research assistant salaries from 85 public and private universities in the United States, along with living wage data for each university location. I discovered that in general, average grad student pay at these universities did not correlate closely with living wage in their areas.

## What's The Problem?
I started thinking about this a few weeks ago, but it appears that no consideration was made in my graduate assistant salary for the cost of living in my town, Fort Collins, CO. For years there has been a trend in this city, and others in Colorado, of people moving in from other areas, far more than those leaving. It makes sense. Colorado is an amazing place to live for so many reasons; natural beauty, healthy residents, 300 days of sun, and the list goes on. This has put some unpleasant pressures on the economy, however, in two very significant ways. The influx of people was so great that the residency vacancy rate plummeted, killing the supply of open houses, and raising home and property prices immensely. 

![Fort Collins in the Spring]({{ site.baseurl }}/images/ft_collins_spring.jpg "Fort Collins in the Spring")
Figure 1. Fort Collins skyline in the Spring, probably. Courtesy of city-data.com.

For example, according to [biggerpockets.com](https://www.biggerpockets.com/renewsblog/2016/02/11/most-and-least-vacancies-us-cities/), Fort Collins had the second lowest vacancy rate in 2016, with only 0.2% of residences open. This drop in supply caused prices to skyrocket. Fort Collins median home prices have risen by more than 50% in the past 10 years, and are continuing to rise. On the other hand, the influx of people has put pressure on the job market, as qualified candidates have flooded the area and inboxes of recruiters. This spoiling for choice has led to an extremely competitive job market, while salaries have no pressure to rise.

But this isn't a post about the woes of the Fort Collinsian economy. This is a post about graduate research assistant (GRA) salaries and the living wage. Basically, I wanted to know if GRA wages in the US tracked with the living wage, and to determine where GRA's have the best and worst deals relative to the area costs of living.

## Some Notes on the Methods
The living wage I'm using comes from [MIT](http://livingwage.mit.edu/), and is the work of Dr. Amy K. Glasmeier. It appears to be a pretty comprehensive attempt at estimating this quantity with a surprisingly fine geospatial granularity in the US. Dr. Glasmeier is careful to make the distinction between living wage and poverty wage. Essentially, the living wage seeks to find the basic required salary needed for an individual or family to meet their basic needs without relying on public assistance. The User Guide from the site puts it thus:

>The living wage is the minimum income standard that, if met, draws a very fine line
between the financial independence of the working poor and the need to seek out public
assistance or suffer consistent and severe housing and food insecurity. In light of this fact, the
living wage is perhaps better defined as a minimum subsistence wage for persons living in the
United States.

I searched too for a "thriving" wage calculator, something that would indicate that the wage earner and their family situation were relatively comfortable, able to own a home, afford to eat out and vacation a couple times a year, and put something back in savings. A cursory search yielded nothing to this effect, and I'd appreciate any examples of such a thing in the comments!

### The Data
So, normally I will go through some programming effort to carefully scrape and clean data in the absence of a pre-made dataset or API. Here, though, I decided a more hands-on approach was appropriate. I pulled the data from [Glassdoor](https://glassdoor.com) by searching their salaries database for "graduate research assistant." This yielded average salaries for institutions with that job title, ordered with highest number of entries first. I wanted to get universities from as many states in the US as possible, while also choosing to double or triple up in high density areas like Boston and New York. I also examined each estimate to ensure that the number of data points was 15 or above, to ensure some decent estimate was provided. I'm not convinced this is either a random sample nor that it is amazingly representative, but given the limitations of the access I have, I think it's a pretty solid dataset.

The living wage site contained the hourly rates for many scenarios. I chose to record the "1 Adult," "1 Adult 1 Child," and "2 Adults" scenarios. The first describes the situation where a grad student needs to support only theirself, and is the sole wage earner for their needs. The second describes a single mother or father scenario, and the third, two working adults, each needing to earn that wage to meet the living wage.

The dataset is [here]({{ site.baseurl }}/images/living_wage_gra_wage_selected_universities_2017.csv), if you want to play with it too.

## Do Universities Care If We Can Survive?
According to this particular data, the answer to that appears to be 'no'. I plotted the average salary for GRAs at each university against that university area's living wage for the "1 Adult" case, and fit a linear trendline to calculate an $$ R^2 $$ value to examine the correlation between the two. If the correlation were high, I would have concluded that it appears that GRA salaries are commensurate with living wage.

![GRA salaries vs living wage]({{ site.baseurl }}/images/gra_pay_vs_living_wage.png "GRA salaries vs living wage")
Figure 2. GRA Salaries vs Living Wage at 85 selected universities in the US.

Given the relatively low correlation value, it might be concluded that while some universities attempt to pay commesurately with cost of living, overall, universities don't seem to notice or care. There is a visible trend, but it is also not a convincing one. Of course in this analysis many confounding variables aren't considered, such as school prestige, controlling for different majors, and experience levels, but as a first pass with available data, this appears to be an interesting result.

## Final Thoughts
In the dataset linked above you will find a ratio that I calculated comparing GRA salary and 1 Adult living wage, in an effort to quantify how well each school did. A value of 1 indicates that the salary is equal to the living wage, and higher values indicate better wages relative to cost of living. The top 5 schools in this dataset by this metric were:

|Rk |   |University            |   | Location       |   | Salary/Living Wage |
|---|---|----------------------|---|----------------|---|--------------------|
|1  |   |Carnegie Mellon       |   | Pittsburgh, PA |   | 1.42               |
|2  |   |St. Louis University  |   | St. Louis, MO  |   | 1.35               |
|3  |   |Notre Dame            |   | South Bend, IN |   | 1.32               |
|4  |   |Washington University |   | St. Louis, MO  |   | 1.31               |
|5  |   |Georgia Tech          |   | Atlanta, GA    |   | 1.28               |

And the bottom 5,

|Rk |   |University            |   | Location         |   | Salary/Living Wage |
|---|---|----------------------|---|------------------|---|--------------------|
|1  |   |Georgetown            |   | Washington, DC   |   | 0.84               |
|2  |   |Massachusetts         |   | Cambridge, MA    |   | 0.85               |
|3  |   |Maryland              |   | College Park, MD |   | 0.87               |
|4  |   |Arkansas              |   | Fayetteville, AR |   | 0.88               |
|5  |   |Vermont               |   | Burlington, VT   |   | 0.91               |

The data in the chart above indicated that universities did not appear to pay in keeping with cost of living, which might lead us to expect that the best salary/living wage ratios occur in areas where the cost of living is lower. In the top 5 table, we see that the best ratios are in lower cost of living areas, with top tier universities. The bottom 5 salaries include two of the highest cost of living areas in the US, in DC and Boston. Burlington, VT, is another expensive area, though I'm not totally sure what's going on with Arkansas, in general or with respect to this analysis. Perhaps salaries are more correlated with university prestige than cost of living?
