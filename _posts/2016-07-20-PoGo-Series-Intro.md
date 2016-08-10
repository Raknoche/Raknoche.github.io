---
layout: post
title: PoGo Series&#58; Introduction
comments: true
---

An introduction to the X part series covering various analysis techniques using data from Pokemon Go.

<!--more-->

Table of Contents:
Put a link to each part of the series here


--

Unless you’ve been living under a rock for the past week, you’ve heard of the new augmented reality game Pokemon Go.  Within two days of release the app was [installed on over 5% of all android devices in the US](https://www.similarweb.com/blog/pokemon-go), more than doubling the installations of other popular applications such as Tinder.  

The game uses your phone’s GPS coordinates to track your real-world location.  As you play, pokemon appear in the world around you, displayed on your phone’s screen through an augmented reality version of your surroundings.
As with all pokemon games, the main objective is to complete your collection of pokemon and “catch ‘em all.”  

The game includes some secondary, player-versus-player objectives as well.  Once your avatar reaches level 10 you choose to join one of three teams — Team Mystic (blue), Team Valor (red), and Team Instinct (yellow).  These teams battle each other for control over Pokemon Gyms that are scattered around the world.  Owning a gym grants access to limited amounts of premium currency which can be used to buy resources in the game.

As you might expect, users have become incredibly passionate about their teams.  Rivalries have sparked thousands of tweets each day which profess loyalty to one’s own team or disparage the others.
 
I thought this would be a great opportunity to explain how to collect data from twitter’s feed and perform in depth analysis on the results.  The plan is simple:

1. Develop an understanding of the Twitter API 
2. Collect tweets that mention each Pokemon Go team using Python's Tweepy library
3. Use Python's Pandas library to perform an inital clean up of the data
4. Develop an understanding of Naive Bayes classifiers
5. Perform a sentiment analysis to determine if a tweet is saying something positive or negative about the team
6. Count the number of positive tweets about each team in each state.  At the same time, measure statistical and systematic errors on our results and produce confidence intervals
7. Use Python's Bokeh and Basemap libraries to produce a map that visualizes the dominance of each team in every state.  

I have organized these steps in a “Pokemon Go Series" of posts.  You'll be able to find the table of contents for the series at the top of each post.
