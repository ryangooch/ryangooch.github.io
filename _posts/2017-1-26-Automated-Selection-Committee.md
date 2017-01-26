---
layout: post
title: Automated Bracket Seeding with Python and Pandas
date: 2017-1-26 16:43:00
---

## Project Summary
Using Python and basketball ranking data, I generate a bracket with a simple algorithm that attempts to "be better" than 
the efforts of a selection committee of tens of people in seeding the competition. You can find the code I used
[here](https://gist.github.com/ryangooch/db17932967f5b624424d557ac3727773) and the bracket for this week
[here]({{ site.baseurl }}/images/bracket2017-01-25.pdf).

## The Problem
It's fairly common knowledge that the NCAA Tournament Selection Committee for Men's College Basketball is 
[terrible](http://www.sbnation.com/college-basketball/2016/3/14/11218966/ncaa-tournament-bracket-selection-committee-tulsa-st-bonaventure-monmouth). 
Every year, there are bizarre inclusions, objectively worthy teams snubbed, and the seeding for those teams 
actually in the field, questionable at best. It is not hard to find examples every year of legitimate complaints
by teams left out, underseeded, or matched with underseeded teams. 

I can pick out a few examples. In 2014, undefeated Wichita St received the overall 1 seed, and sat near the top of many of the
objective computer ranking systems available at the time. The committee rewarded them for their amazing season with 
a second-round matchup with Kentucky, a team that struggled somewhat throughout the year but that computer rankings placed
[around a 3/4 seed line](http://kenpom.com/index.php?y=2014). The committee for reasons known only to themselves had
given Wichita St an incredibly unfair second round matchup that, unfortunately for the Shockers, they lost, in a game
that NBC Sports called ["the Game of the Year"](http://collegebasketball.nbcsports.com/2014/03/23/wichita-state-loses-kentucky-in-the-game-of-the-year-video/)

![Kentucky defeats Wichita St]({{ site.baseurl }}/images/wichita-state-final-3-against-kentucky.gif "Kentucky defeats Wichita St as shot is missed at the buzzer")

Why was the Game of the Year played in the Second Round, when both teams were Final Four caliber teams? Kentucky of course
would go on to reach the Championship game before finally falling to 7 seed UConn, in a game featuring the 
tournament's highest combined seed matchup for the championship in its history. Yes, the tournament is random. Yes, there are
upsets. But perhaps those two teams deserved to be seeded higher in the first place?

That is the question that spurred this process; **If computers determined the seeding, would the bracket be more fair,
more objectively balanced?**

It is difficult to quantify what "doing better" than the committee looks like, but we have a few decent starting points.
One of the common problems that the committee points to is that they have to analyze the resumes of many teams and come 
to agreement about them over the course of one weekend. There are practices they must try to follow, such as limiting
first and second round rematches of teams who have met in the regular season, in addition to the complex problem of 
comparatively ranking the teams.

We have, thus, a few things to work with here. An algorithm can be much faster than a weekend's worth of time. An algorithm can compare many
criteria and work across many dimensions to find rankings that satisfy those criteria very quickly. An algorithm can fairly
place teams on a bracket, and can even check to make sure they've not played. And, it can even to some degree meet the "eye test",
taking human poll data (ESPN Coaches, AP) into account to make decisions.

## The Data

There has been an explosion in sports ranking systems in recent years, with some respected sources in particular emerging
from the pack:

 * [Kenpom.com](http://kenpom.com/), which relies on team efficiencies and tempos
 * [ESPN's Basketball Power Index (BPI)](http://www.espn.com/mens-college-basketball/bpi), which measures team strength
 accounting for strength of record, pace, site, travel distance, and a few other factors around the circumstances of play
 * [Sagarin](http://www.usatoday.com/sports/ncaab/sagarin/), based on scores of games and recent performance
 
In fact, [Bracket Matrix](http://bracketmatrix.com/) has **82** separate sources of brackets that it aggregates, all of 
them with differing methods for team selection, seeding, etc. Our goal is not to supplant these with a super bracket,
but to simply explore the possibility of a simple, automated algorithm that can "do better" than the committee. 
 
There are a few options for where and how to get data. We can:

1. Scrape rankings from web pages, such as kenpom, BPI, etc, or;
2. We can get a composite rankings from someone who has done this work already.
 
When I first started this project, I opted for method 1. Scraping the pages proved to be fairly straightforward using
Python's urllib3 and BeautifulSoup packages. However, one problem that scaled the problem difficulty up by orders of
magnitude was that each site had differing names for the same colleges. One example was that of Middle Tennessee State
University. Some names that were encountered:
 
* Middle Tennessee State
* Mid Tenn St
* MTSU
* Mid Tennessee St
* etc
 
I briefly explored string matching using [Levenshtein Distance](https://en.wikipedia.org/wiki/Levenshtein_distance), via the
capable and very handy python package [fuzzywuzzy](http://chairnerd.seatgeek.com/fuzzywuzzy-fuzzy-string-matching-in-python/). 
This seemed solid at first, but broke down when teams like Colorado State, Cal State-whatever, and Coppin State all matched
well with CSU. Another approach that came to mind was creating a bag-of-words and using a classifier to make decisions, but
that was prone to error and not as satisfactory as I hoped. 
 
It was at this point I reached out to [Ken Massey](http://www.masseyratings.com/), the statistician that aggregates scores and ranking systems for several
sports, and generates his own ranking system. He was kind enough to reply and told me that he has, over the past 20
or so years, generated a database containing every spelling, misspelling, and abbreviation for every Division 1 school.
This approach has netted him a resource to draw from in aggregating these ranks, but this was outside the scope
of this work.
 
In the end, I chose to take the second approach above, and simply use the data that Ken puts out on the web every week
during the season, his [college basketball composite](http://www.masseyratings.com/cb/compare.htm).
 
## The Code
A quick note on the code. I plan on putting things in classes, cleaning up, and adding functionality, but for now, you can
grab it in script form from the Jupyter Notebook. I hope to get this up in a repo soon, however. You can grab the python
file [here](https://gist.github.com/ryangooch/db17932967f5b624424d557ac3727773) if you like.

First, we import the necessaries, download the csv, and pull the data into memory.
``` python
from urllib.request import urlretrieve
import numpy as np
import pandas as pd
from openpyxl import Workbook, load_workbook
import csv
import datetime

import warnings
warnings.filterwarnings('ignore')

# Get the composite csv from masseyratings.com
urlretrieve('http://www.masseyratings.com/cb/compare.csv', 'masseyratings.csv')

# need to parse the csv
# first row of interest starts at "team" in column 1
# data starts two rows after
data = []
with open('masseyratings.csv', newline='') as csvfile:
    reader = csv.reader(csvfile, delimiter = ',')
    for row in reader:
        data.append(row)
```

The csv file has a header containing information about the ranking systems included, URLs, and then a table of the data,
which have team names, records, a few statistics, and then all the ranking systems. The ranking systems change every week
based on availability, so I didn't bother selecting any, and instead chose to take the median computer rank, and the 
average human poll rank, for each team, before performing a simple weighted average to determine the final average rank for
each team. But first, we must get the dataset into a form we can manipulate.

``` python
# The csv file wasn't formatted in a way pandas could read. 
# There is a header section followed by the data, but the header
# section is variable in length. We can simply iterate through 
# until we see the 'Team' string indicating the start of the data

i = 0 # counter of lines until Team found
for row in data:
    if row and row[0] != 'Team': 
        i = i + 1
    if row and row[0] == 'Team': 
        break

# The header data contains the abbreviations and URLs for the
# various ranking systems in the dataset, might be useful later
header_data = data[:i+2]
team_data = data[i+3:]
column_names = team_data[0] # Dataset column names
# strip whitespace from the column names for normalization
for i in range(len(column_names)):
    column_names[i] = column_names[i].lstrip()

# drop the data set in a pandas Dataframe
team_data_df = pd.DataFrame(team_data[2:],columns=column_names)

# Drop columns that are unnecessary 
cols_to_drop = ["WL","Rank","Mean","Trimmed","Median","StDev","AP","USA"] # columns we won't use
# Split off human ratings and computer ratings
comp_ratings = team_data_df.drop(cols_to_drop,axis=1) # computer ratings
human_ratings = team_data_df[['Team','Conf','AP','USA']] # pulls out human ratings
```

As you can see, we're using pandas here for the heavy lifting of dataset manipulation in Python. I really like using
it as it can easily slice the data, preserve header information, and is pretty fast if used properly. Since these
datasets were fairly small, I decided to opt for ease of coding and created a few extra pandas Dataframes along the way.
The approach I took was to split up the rankings into human and computer, apply some processing to each, and manipulate
each mathematically, before putting the summary statistics of interest in another dataframe.

``` python
summary_df = team_data_df[["Team","Conf"]] # this will hold the final results

human_ratings['AP'] = human_ratings['AP'].str.strip()
human_ratings['USA'] = human_ratings['USA'].str.strip()
human_ratings = human_ratings.apply(lambda x: pd.to_numeric(x, errors='ignore'))
comp_ratings = comp_ratings.apply(lambda x: pd.to_numeric(x, errors='ignore'))

summary_df["comp_mean"] = comp_ratings.mean(axis=1,numeric_only=True)
summary_df["human_mean"] = human_ratings.mean(axis=1,numeric_only=True,skipna=True)

def rank_calc(x, y):
    if np.isnan(y) :
        return x
    else:
        return ((2 * x + y) / 3.)

summary_df["final_rank"] =     np.vectorize(rank_calc)(
        summary_df["comp_mean"], summary_df["human_mean"])

summary_df.sort_values(by=['final_rank'],inplace=True)
```

Above, I create a function called rank_calc to perform the weighted average, then use numpy's vectorize function
to apply it to the dataframe to calculate the final rank, before sorting by that value.

At this point, we must consider the rules of selection for the NCAA tournament. 32 teams receive automatic bids
into the tournament, as winners of their respective conferences, and the remaining teams are filled "at-large". In
other words, the remaining 36 teams are filled by the best 36 that did not get an automatic bid into the tournament.

In order to accomplish this selection, I used the groupby functionality of the dataframe to find the top ranked team
in each conference, then pulled those teams out into their own list. Then, I did the same for the remaining teams by 
indexing the summary dataframe by the opposite query of the auto bid teams, selecting all teams outside that group.

Finally, I created a dataframe to store the final 68 teams.

``` python
# Using groupby to grab the auto bids
auto_bid_teams = summary_df.groupby(['Conf']).head(1)['Team'].values

# and we can use ~isin now to get at larges
at_large_teams = summary_df[~summary_df['Team'].isin(auto_bid_teams)].head(36)['Team'].values

# all 68 teams in one array
all_68 = np.append(auto_bid_teams,at_large_teams)

final_68 = summary_df[summary_df['Team'].isin(all_68)]
```

This is where things become a little less automated unfortunately. Since the NCAA Tournament has playin teams, we
must consider extra seeds at the 11 and 16 spots. Recall that there are essentially 4 sets of seeds 1-16. It would
have been simple to generate a list of seed labels with which to append to the dataframe, but I opted for a manual 
approach for easier visualization.

Seeding the bracket itself is something that needs work. I probably could have spent a few more hours, created a bracket
graphic, then placed all the teams on it. Instead, I opted to use the openpyxl package to place teams on specific cells
corresponding to exact bracket placements, then copy those names to a bracket template I set up in Excel. This workflow is
not automated, but it adds only as much time as a copy and paste requires, so I'm OK with it for now for this project.

The committee has stated it does not use the Snake method for seeding the bracket, but here, I chose to use it as it was
simple and fair (ie, the best 1 seed is in the same region as the worst 2, best 3, worst 4, and so on). In code, this looks
like:

``` python
#add seeds
seeds = np.array([
        1,1,1,1,
        2,2,2,2,
        3,3,3,3,
        4,4,4,4,
        5,5,5,5,
        6,6,6,6,
        7,7,7,7,
        8,8,8,8,
        9,9,9,9,
        10,10,10,10,
        11,11,11,11,11,11,
        12,12,12,12,
        13,13,13,13,
        14,14,14,14,
        15,15,15,15,
        16,16,16,16,16,16
    ])


# In[25]:

final_68["seed"] = seeds


# In[26]:

final_68.head(16)


# In[27]:

# Need to tell Excel which cells get which team. Because we're 
# using 'snake' method, there are specific cells corresponding
# to each seed and rank. Easiest way to do this is to simply
# hard code the table in, for now
excel_placements = [
    'C7', 'R7', 'R39', 'C39', 'C67', 'R67', 'R35', 'C35', 
    'C27', 'R27', 'R59', 'C59', 'C51', 'R51', 'R19', 'C19',
    'C15', 'R15', 'R47', 'C47', 'C55', 'R55', 'R23', 'C23',
    'C31', 'R31', 'R63', 'C63', 'C43', 'R43', 'R11', 'C11',
    'C13', 'R13', 'R45', 'C45', 'C65', 'R65', 'R33', 'C33',
    'C25', 'R25', 'Q71', 'Q73', 'D71', 'D73', # 11 seeds
    'C49', 'R49', 'R17', 'C17', 'C21', 'R21', 'R53', 'C53',
    'C61', 'R61', 'R29', 'C29', 'C37', 'R37', 'R69', 'C69',
    'C41', 'R41', 'O71', 'O73', 'F71', 'F73'
]

# append that to our final_68 dataframe
final_68['excel'] = excel_placements

for index, row in final_68.iterrows():
    print(row['Team'], row['excel'])

wb = Workbook()
ws = wb.active
for index, row in final_68.iterrows():
    ws[row['excel']] = row['Team']

# get today's date, save bracket
today = str(datetime.date.today())
save_file = 'bracket' + today + '.xlsx'
wb.save(save_file)
```

And we're done! The example pdf for this week can be found [here]({{ site.baseurl }}/images/bracket2017-01-25.pdf)

## Future Work

I really want to automate the bracket generation in future. As I said above as well, I want to wrap this functionality 
in a few classes, make it more efficient and pythonic, and allow for users to select which rankings they want to 
include and specify a custom algorithm for calculating the final rank. I'd love to re-visit the scraping one day too, 
and build that database of team names.

One more substantial ambitious project I'd love to try out is to scrape game data, team data, players, and place it all
in a graph database. This would allow for more complex algorithms to determine comparative ranks, and might allow for 
better prediction of game outcomes, along with assisting coaches in making decisions pre-game. These types of networks are
great for finding "sub-networks", and I envision a possibility to compare factors in losing efforts vs winning efforts to
better prepare for new opponents. Hopefully you will see that tutorial go up soon!

Ryan

