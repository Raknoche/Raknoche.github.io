---
layout: post
title: Fantasy Football Predictions &mdash; A First Look
comments: true
---

A sneak peak at using machine learning to predict fantasy football scores.

<!--more-->

{% include math.html %}

Fantasy Football season is coming up, and with it comes millions of people with a vested interest in football data.  This year, I've been working on a machine learning approach to predict the fantasy points of each NFL player on a rest-of-season and week-by-week basis.   Projections are made using a [Support Vector Machine (SVM)](http://cs229.stanford.edu/notes/cs229-notes3.pdf) with features taken from each player's, team's, and opponent's stats over the past 17 regular season games. Games which a player did not participate in are not considered for that player's average.


# Prediction Performance

I set aside a random sample of data from the 2009-2015 NFL seasons, as well as the entirety of the 2014 season to evaluate the prediction model's performance.  The SVM was not tuned to these test data sets in any way, so they should be a good indication of the model's performance.  I used a number of metrics during the performance evaluation, the easiest to explain being [Pearson's coefficient of determination](https://en.wikipedia.org/wiki/Coefficient_of_determination), commonly known as $$R^2$$.  An $$R^2$$ value of one means that 100% of the data's variation is explained by the model, while a $$R^2$$ of zero means that the model's predictions are not correlated with the real outcomes in any way. 

Here are the $$R^2$$ results for the model for the sequestered 2014 season, using PPR scoring.


|                            | QB   |   WR |  RB  |   TE | 
|:--------------------------:|:----:|:----:|:----:|:----:|
| Season Projections         | 0.98 | 0.96 | 0.95 | 0.97 | 
| Initial Season Projections | 0.67 | 0.72 | 0.54 | 0.71 |
| Week-by-Week Projections   | 0.40 | 0.51 | 0.52 | 0.47 |

In the above table, "Season Projections" refers the performance of the model as the NFL stats update throughout the season. In this case, the stat averages are updated on a weekly basis and are used to predict the next week's outcome.  This allows the model to improve it's predictions as the season plays out, and results in extremely high performance ($$R^2>0.95$$ for all positions) on the total points predicted for each player at the end of the season.

The "Initial Season Projections" refer to the season-long projections using only information that is available at the beginning of the 2014 season.  In other words, all of the stat averages come from the previous season and are projected throughout the entire 2016 season.  This is the metric to use when evaluating the performance of our model for the 2016 draft at the start of the fantasy football season.  We see that the model's performance varies depending on the player's position.  It can explain 72% of the variation in season-long wide receiver performance, 71% of the variation in season-long tight end performance, 67% of the variation in season-long quarterback performance, and 54% of the variation in season-long running back performance.

The "Week-by-Week Projections" refer to the performance of our model on a week-by-week basis.  Player's often underperform or over perform during a particular game, so the $$R^2$$ values of our model are much lower on a week-by-week basis than a season-long basis. Still, our model's week-by-week average $$R^2$$ value of about 0.475 is better than the many professional sites projections. ([Source from FantasyFootballAnalytics](http://fantasyfootballanalytics.net/2016/03/best-fantasy-football-projections-2016-update.html))



To visualize the meaning of each $$R^2$$ value, I've included a few plots below.  The first plot shows the "Season Projections" of wide receivers versus the actual season total in a PPR format.  This data has $$R^2=0.96$$.

![png](https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/FF_PPR/SeasonProjectionWR.png)

Here's a similar plot of the "Initial Season Projections" of wide receivers versus the actual season total in a PPR format.  This data has an $$R^2=0.72$$.


![png](https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/FF_PPR/InitialSeasonProjectionWR.png)

# Projections for the 2016 Draft




At the bottom of this page I've included my initial projections for the 2016 fantasy football season, using both standard scoring and point-per-reception (PPR) formats.  Before you take a look, you should be aware of a few notes:




1. The machine learning predictions requires that a player has played at least one game in the past 17 regular season matches.  If this isn't the case, there will not be any projections for that player.  In particular, first-year rookies or returning retired players (such as Josh Gordon) will not appear on the projections.  Players who have only played a handful of games recently will have poorly-measured averages, and are more likely to have incorrect predictions.


2. The projections do not consider fantasy news such as player suspensions, recent injuries, or coaches' statements.  You should view these projections as a measure of how well a player is expected to perform if they play the entire season without injury, and in a similar role to last season.  For instance, if a player was a starting running back last season, and might be in a running-back-by-committee this season (I'm looking at you, Devonta Freeman) it is likely their projections will be too high.  Likewise, if a player is injured going into the 2016 season and will have to take it easy the first few games (for instance, Jamal Charles) their projected season total is also likely too high.



 
# <u>Standard Scoring Predictions</u>
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

# <u>PPR Predictions</u>

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