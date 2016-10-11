---
layout: post
title: Fantasy Football Predictions &mdash; A First Look
comments: true
---

A sneak peak at using machine learning to predict fantasy football scores.

<!--more-->

{% include math.html %}

Fantasy Football season is coming up, and with it comes millions of people with a vested interest in football data.  This year, I've been working on a machine learning approach to predict the fantasy points of each NFL player on a rest-of-season and week-by-week basis.  

Projections are made using a [Support Vector Machine (SVM)](http://cs229.stanford.edu/notes/cs229-notes3.pdf) with features taken from each player's, team's, and opponent's stats over the past 17 regular season games. Games which a player did not participate in are not considered for that player's average.

The model is still a work in progress, but it's working well enough to release season projections before the regular season games start.  As usual, when I've finished polishing the project I'll do a series of blog posts explaining the process in detail. 

# Prediction Performance

I set aside a random sample of data from the 2009-2015 NFL seasons, as well as the entirety of the 2014 season to evaluate the prediction model's performance.  The SVM was not tuned to these test data sets in any way, so they should be a good indication of the model's performance.  I used a number of metrics during the performance evaluation, the easiest of which to explain being [Pearson's coefficient of determination](https://en.wikipedia.org/wiki/Coefficient_of_determination), commonly known as $$R^2$$.  An $$R^2$$ value of 1.0 means that 100% of the data's variation is explained by the model, while an $$R^2$$ value of zero means that the model's predictions are not correlated with the real outcomes in any way. 

Here are the $$R^2$$ results for the current model on the sequestered 2014 season, using PPR scoring.


|                            | QB   |   WR |  RB  |   TE | 
|:--------------------------:|:----:|:----:|:----:|:----:|
| Season Projections         | 0.98 | 0.96 | 0.95 | 0.97 | 
| Initial Season Projections | 0.67 | 0.72 | 0.54 | 0.71 |
| Week-by-Week Projections   | 0.40 | 0.51 | 0.52 | 0.47 |

In the above table, "**Season Projections**" refers the performance of the model as the NFL stats update throughout the season. In this case, the stat averages are updated on a weekly basis and are used to predict the next week's outcome.  This allows the model to improve it's predictions as the season plays out, and results in extremely high performance ($$R^2>0.95$$ for all positions) on the total points predicted for each player at the end of the season.

The "**Initial Season Projections**" refer to the season-long projections using only information that is available at the beginning of the 2014 season.  In other words, all of the stat averages come from the previous season and are projected through the end of the 2014 season.  This is the metric we should use when evaluating the performance of our model for the 2016 draft.  We see that the model's performance varies depending on the player's position, explaining 72% of the variation in season-long wide receiver performance, 71% of the variation tight end performance, 67% of the variation quarterback performance, and 54% of the variation in running back performance.

The "**Week-by-Week Projections**" refer to the performance of our model on a week-by-week basis.  Player's often underperform or over perform during a particular game, so the $$R^2$$ values of our model are much lower on a week-by-week basis than a season-long basis. Still, our model's week-by-week average $$R^2$$ value of 0.475 is better than the many professional sites projections. ([Source from FantasyFootballAnalytics](http://fantasyfootballanalytics.net/2016/03/best-fantasy-football-projections-2016-update.html))



To visualize the meaning of each $$R^2$$ value, I've included a few plots below.  The first plot shows the "Season Projections" of wide receivers versus the actual season total in a PPR format.  This data has $$R^2=0.96$$.

![png](https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/FF_PPR/SeasonProjectionWR.png)

Here's a similar plot of the "Initial Season Projections" of wide receivers versus the actual season total in a PPR format.  This data has a lower $$R^2$$ of $$0.72$$ since all of the statistics come from the previous season.

![png](https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/FF_PPR/InitialSeasonProjectionWR.png)

# Projections for the 2016 Draft




At the bottom of this page I've included my initial projections for the 2016 fantasy football season, using both standard scoring and point-per-reception (PPR) formats.  Before you take a look, you should be aware of a few notes:




1. The machine learning predictions require that a player has played at least one game in the past 17 regular season matches.  If this isn't the case, there will not be any projections for that player.  In particular, first-year rookies or returning retired players (such as Josh Gordon) will not appear on the projections. 

2. The projections do not consider fantasy news such as player suspensions, recent injuries, or coaches' statements.  

3. I use a clustering algorithm to extract natural tiers of players from the projections. The idea comes from [Boris Chen's](http://www.borischen.co/) use of clustering in his aggregation of expert's predictions.


You should not treat these projections as the the final word on who you should draft.  Instead, view them as a measure of how well a player is expected to perform if they play the entire season without injury, and in a similar role to last season.  The projections are particularly useful if you are trying to determine how well a player compares to the rest of the league, since each player's performance is quantified with a projected score rather than a simple ranking.  For instance, you can see exactly how much better a player like Antonio Brown is expected to be compared to the rest of the wide receivers.

If you do use these projections, be sure to look up recent news on a player before drafting them.  It's entirely possible that a player is injured or expected to play a weaker role this season, which would not be reflected in the projections.  For instance, if a player was a starting running back last season, and might be in a running-back-by-committee this season (I'm looking at you, Devonta Freeman) it is likely their projections will be too high.  Likewise, if a player is injured going into the 2016 season and will have to take it easy the first few games (like Jamal Charles) their projected season total is also likely too high. 

**Update (10/10/16)**:  I've decided to spin this project into a stand alone web app that will include fantasy projections as well a various other fantasy related analyses.  I believe the project has grown too large to describe in a series of blog posts, so I won't be posting any follow ups to this analysis in the near future.  For the curious, you can find the code used to produce these plots on my [github page](https://github.com/Raknoche/MyCode/tree/master/CodeForDealingData/FantasyFootballPredictions), which is linked in the sidebar.  Be warned &mdash; the code is messy and poorly commented, since I put it together in a rush before the start of the season and have not revisited it since.

<br>

 
# <u>Standard Format Predictions</u>
<script language="javascript"> 
function toggle(original,text,tog) {
	var ele = document.getElementById(tog);
	var text = document.getElementById(text);
	if(ele.style.display == "block") {
    		ele.style.display = "none";
		text.innerHTML = original;
  	}
	else {
		ele.style.display = "block";
		text.innerHTML = "Hide " + original;	}
} 
</script> 
 
<a id="WRText" href="javascript:toggle('WR Predictions (Standard)','WRText','WRTextTog');">WR Predictions (Standard)</a>
<div id="WRTextTog" style="display: none"><img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/FF_Standard/WR_Predictions.png"></div>

<a id="RBText" href="javascript:toggle('RB Predictions (Standard)','RBText','RBTextTog');">RB Predictions (Standard)</a>
<div id="RBTextTog" style="display: none"><img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/FF_Standard/RB_Predictions.png"></div>

<a id="TEText" href="javascript:toggle('TE Predictions (Standard)','TEText','TETextTog');">TE Predictions (Standard)</a>
<div id="TETextTog" style="display: none"><img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/FF_Standard/TE_Predictions.png"></div>

<a id="QBText" href="javascript:toggle('QB Predictions (Standard)','QBText','QBTextTog');">QB Predictions (Standard)</a>
<div id="QBTextTog" style="display: none"><img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/FF_Standard/QB_Predictions.png"></div>

<a id="FLEXText" href="javascript:toggle('FLEX Predictions (Standard)','FLEXText','FLEXTextTog');">FLEX Predictions (Standard)</a>
<div id="FLEXTextTog" style="display: none"><img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/FF_Standard/FLEX_Predictions.png"></div>

<br><br>

# <u>PPR Format Predictions</u>

<a id="WRTextPPR" href="javascript:toggle('WR Predictions (PPR)','WRTextPPR','WRTextTogPPR');">WR Predictions (PPR)</a>
<div id="WRTextTogPPR" style="display: none"><img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/FF_PPR/WR_Predictions.png"></div>

<a id="RBTextPPR" href="javascript:toggle('RB Predictions (PPR)','RBTextPPR','RBTextTogPPR');">RB Predictions (PPR)</a>
<div id="RBTextTogPPR" style="display: none"><img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/FF_PPR/RB_Predictions.png"></div>

<a id="TETextPPR" href="javascript:toggle('TE Predictions (PPR)','TETextPPR','TETextTogPPR');">TE Predictions (PPR)</a>
<div id="TETextTogPPR" style="display: none"><img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/FF_PPR/TE_Predictions.png"></div>

<a id="QBTextPPR" href="javascript:toggle('QB Predictions (PPR)','QBTextPPR','QBTextTogPPR');">QB Predictions (PPR)</a>
<div id="QBTextTogPPR" style="display: none"><img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/FF_PPR/QB_Predictions.png"></div>

<a id="FLEXTextPPR" href="javascript:toggle('FLEX Predictions (PPR)','FLEXTextPPR','FLEXTextTogPPR');">FLEX Predictions (PPR)</a>
<div id="FLEXTextTogPPR" style="display: none"><img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/FF_PPR/FLEX_Predictions.png"></div>

