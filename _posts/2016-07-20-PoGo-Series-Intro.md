---
layout: post
title: PoGo Series&#58; Introduction
comments: true
---

An introduction to the seven part series covering various analysis techniques using data from Pokemon Go.

<!--more-->

Table of Contents:

1. [Introduction to the Series](http://raknoche.github.io/2016/07/20/PoGo-Series-Intro/)
2. [The Twitter API](http://raknoche.github.io/2016/07/21/PoGo-Series-TwitterAPI/)
3. [The Tweepy Library](http://raknoche.github.io/2016/07/23/PoGo-Series-Tweepy/) 
4. [Naive Bayes Classifiers](http://raknoche.github.io/2016/07/24/PoGo-Series-NaiveBayesClassifier/)
5. [Training a Sentiment Analyzer](http://raknoche.github.io/2016/07/28/PoGo-Series-Sentiment-Analyzer/)
6. [Statistical Analysis of the Data](http://raknoche.github.io/2016/07/31/PoGo-Series-Statistical-Analysis-of-the-Data/)
7. [Visualizing the Data with Choropleth Maps](http://raknoche.github.io/2016/08/03/PoGo-Series-Making-a-Choropleth-Map/)


---------------------


Unless you’ve been living under a rock for the past week, you’ve heard of the new augmented reality game Pokemon Go.  Within two days of release the app was [installed on over 5% of all android devices in the US](https://www.similarweb.com/blog/pokemon-go), more than doubling the installations of other popular applications such as Tinder.  

The game uses your phone’s GPS coordinates to track your real-world location as you play.  Pokemon appear in the world around you, displayed on your phone’s screen through an augmented reality version of your surroundings.
As with all Pokemon games, the main objective is to complete your collection of Pokemon and “catch ‘em all.”  

The game includes some secondary, player-versus-player objectives as well.  Once your avatar reaches level 10 you choose to join one of three teams — Team Mystic (blue), Team Valor (red), or Team Instinct (yellow).  These teams battle each other for control over Pokemon Gyms that are scattered around the world.  Owning a gym grants access to limited amounts of premium currency which can be used to buy resources in the game.

As you might expect, users have become incredibly passionate about their teams.  Rivalries have sparked thousands of tweets each day which profess loyalty to one’s own team or disparage the others.
 
I thought this would be a great opportunity to explain how to collect data from Twitter’s feed and perform in depth analysis on the results.  In this series of posts, we'll map the dominance of each Pokemon Go team in each of the US states.  We'll use the fraction of tweets which reference each team in a positive manner as a measure of the team's dominance.  The plan is simple:

1. Develop an understanding of the Twitter API 
2. Collect tweets that mention each Pokemon Go team using Python's Tweepy library
3. Develop an understanding of Naive Bayes Classifiers
4. Use a Naive Bayes Classifier to determine if each of the tweets we collected has a positive or negative tone.  Use this information to remove any negatively toned tweets from our collection.
5. After cleaning the data, count the number of positive tweets about each team in each state.  Use the fraction of tweets about a particular team as a measure of the dominance of that team in each state. 
6. Use Python's Bokeh and Basemap libraries to produce a map that visualizes our results.

Before we get into the details of the analysis, here's a sneak peak of the final result.

<div class="bokehTable">
{% include PoGo_TeamDominance.html %}
</div>