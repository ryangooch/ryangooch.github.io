---
layout: post
title: Automated Bracket Seeding with Python and Pandas: An Update
date: 2017-01-31 14:00:00
tags: python, pandas, ncaa, ncaa tournament, bracket
---

# Automated Bracket Seeding With Python and Pandas: An Update

## Project Summary

In my last post, I ended with some pie-in-the-sky hare-brained ideas about actually returning to previous work, cleaning up code, and adding functionality. It must be a blue moon because I did it! You can now get the python utility to make your own NCAA brackets [here](https://github.com/ryangooch/automated_ncaa_bb_bracket)! By the way, [my latest bracket]({{ site.baseurl }}/images/bracket2017-01-30.pdf), if you're interested.

## Brackets at a Few Clicks of a Button

I have usage instructions on the github page, but I wanted to expand a little bit on what I'd done. Basically, I moved all the functionality into an easy-to-use python class called ` Bracketeer ` in `bracket_maker.py` at the above repo. The utility follows the workflow of the script essentially, with a download of data, sanitization, calculation of ranks, and saving to Excel workbook.

I chose to expose a few steps to allow some customization. The big one is that the user can now choose whichever computer polls they prefer, and really refine the type of metric they want to build.

 I think there are really two schools of thought on this point, however. 
 
 * I'm tempted to use the statistics approach, and do a sort of ensemble approach of the ranks; all ranking metrics included have some ability to access the "truth" (here, the true rank, if such a thing exists), so why not take some combination of all of them? Won't the noise cancel out, leaving the best estimate from the sample? Well, maybe. 
 * I think the other school of thought is that a curated list of rnaking metrics is best. Carefully choose rankings that all try to extract information about differing important aspects. For example, Kenpom, which attempts to find a predictive comparison of all teams, regardless of circumstance, while ESPN's BPI is all about examining circumstances surrounding teams and games. Hence, these two may get at two crucially important aspects without too much overlap.
 
 For now, I will simply take all rankings and hope that the noise is ironed out. I actually do really like the latest bracket; it seems like what an ideal committee would choose, especially given the recent losses of several top teams.
 
## Future Work
I want to really explore in more depth ranking systems and perhaps curate a few subsets of ranks that I really like. Also, it would be nice if arbitrary summary statistics could be calculated for the computer ranks, instead of relying simply on the mean. One thing I have done in the past is to simply discard the highest and lowest rank each team receives, then average the remaining ranks. This makes a lot of sense if the sample size is smaller. When there are 40 teams, perhaps removing the highest and lowest 10% makes more sense. I'd like to implement that though.

Maybe this belongs in the fantasy realm, but a ranking system I'd like to explore is one that tries to find groupings of teams. This isn't a terribly new idea, although I don't know of anyone building rankings in this way. We've seen Jimmy Dykes' plane, around tournament time. He puts a few teams in "First Class," some in "Business," some in "Coach," or something like that. Essentially admitting that you can't reliably say any one of the teams is objectively better than the others. Well, I think that's a fascinating concept numerically. I want to due some k-means analysis to see if indeed subsets of teams emerge, and where they fall. But, that is for another day.
