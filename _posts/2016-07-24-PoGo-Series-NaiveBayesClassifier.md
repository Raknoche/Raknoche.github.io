---
layout: post
title: PoGo Series&#58; Introduction to Naive Bayes Classifiers
comments: true
---




A discussion of Bayes theorem and how it is used in Naive Bayes Classifiers.

<!--more-->

TABLE OF CONTENTS


Welcome back to our on-going Pokemon Go analysis series.  In the previous post, we used Python's Tweepy package to collect tweets about each Pokemon Go team and store them in a text file.  Some of the tweets mention the pokemon go teams in a positive light, while others denegrate the team that's mentioned.  We'll be training a Naive Bayes Classifier to differentiate these two classes of tweets, but before we do that, we need to understand what a Naive Bayes Classifier is.

In this post, we'll discuss the following topics:

1. [Conditional Probability](#conditionalprob)
2. [The Law of Total Probability](#totalprob)
3. [Bayes' Theorem](#bayes)
4. [Naive Bayes Classifiers](#naiveBayes)
5. [Applying the Naive Bayes Classifier](#naiveEx)

# <a name="conditionalprob"></a> Conditional probability

Suppose we want to calculate the probability of our favorite sports team winning their next match.  If we only had information about the team’s win-loss record, we could estimate their chances of winning as the number of games they have won divided by the number of games they have played.  Mathematically, if $$\mathsf{W}$$ represents the team winning, and $$\mathsf{L}$$ represents the team losing, this is written:

$$\mathsf{ P(W) = \frac{W}{W + L}} \tag{1}$$

Now, suppose that we obtained additional information about each time our team won and lost a game.  For instance, maybe we kept track of whether our team was playing at home or not.  Now we can ask more complex questions, such as “What is the probability of our favorite team winning, given that they are at home?”  This is known as a **conditional probability**.  We can calculate the answer by counting the how often our team wins while at home, and dividing by the total number of times our team played at home (regardless of the outcome).  Mathematically, if $$\mathsf{H}$$ represents the team playing at home:

<a name="eq2"></a>

$$\mathsf{P(W \lvert H) = \frac{ P( W \cap H)}{P(H)}}\tag{2} $$ 

A few notes about the above equation:

* $$\mathsf{P(W \; \lvert \; H)}$$ is the conditional probability that our team will win, given that they play at home.  The \"$$\lvert$$\" symbol is used to declare the conditions that are **given**.
* $$\mathsf{P( W \cap H)}$$ is the probability of our team winning and playing at home.  The \"$$\cap$$\" symbol is used to denote the **intersection** where both of these events occur.
* $$\mathsf{P(H)}$$ is the probability of our team playing at home.
* The probability of our team winning without specifying any conditions, $$\mathsf{P(W)}$$ is known as the **prior probability** of $$\mathsf{W}$$.

<a name="goback"></a>

In words, [Equation 2](#eq2) is stating that the probability of $$\mathsf{A}$$ occuring given $$\mathsf{B}$$ has occured ($$\mathsf{P(A \lvert B)}$$ is equal to the probability of $$\mathsf{A}$$ and $$\mathsf{B}$$ both occuring ($$\mathsf{P( W \cap H)}$$) divided by the probability of $$\mathsf{B}$$ occuring.


# <a name="totalprob"></a> The Law of Total Probability

Now, suppose that we knew how often our favorite team won when playing at home, and we knew how often they won while playing away, but we neglected to keep track of how often the team won in general, regardless of where they played.  The **Law of Total Probability** states how to turn a set of conditional probabilities into a prior probability.  In general, if our sample space is partioned into $$n$$ different outcomes, then for some event $$\mathsf{A}$$

$$\mathsf{P(A) = \sum_{i=1}^n P(A \cap B_i) = \sum_{i=1}^n P(A \lvert B_i)P(B_i)}\tag{3}$$

where we have used [Equation 2](#eq2) to write $$\mathsf{P(A \cap B_i)}$$ in terms of conditional probabilities.  Applying this to our favorite team's record, the law is simply stating that the probability of our team winning in general is equal to the probability of our team winning and being at home, plus the probability of our team winning and being away:

$$\mathsf{P(W) = P(W \cap H) + P(W \cap not \; H)= P(W \lvert H)P(H) + P(W \lvert not \; H)P(not \; H)}\tag{4}$$

If you need an intuitive explanation for the Law of Probabilities, imagine that our team played 100 games total.  Of those 100 games, they won 40 out of 50 games at home, and 30 out of 50 games while away.  The total number of games they won is clearly 70.  The fraction of games that our team won overall is then 70/100, which you can obtain from the Law of Total Probability by adding the fraction of games where our team won and were at home to the fraction of games that our team won and were away:  40/100 + 30/100 = 70/100. 

A closing note for this section &mdash; do not confuse the probability of our team winning and being at home with the probability of our team winning while at home.  The former is the probability of both events happening if we randomly sample from any game, while the later is the probability of winning, given that we are only sampling from the games played at home.


# <a name="bayes"></a> Bayes' Theorem

Finally, we can discuss the foundation of Naive Bayes Classifiers &mdash; Bayes' theorem.  Consider the general form of [Equation 2](#eq2), for some event $$\mathsf{A}$$ and some given condition $$\mathsf{B}$$:

<a name="eq5"></a>

$$\mathsf{P(A \lvert B) = \frac{ P( A \cap B)}{P(B)}}\tag{5} $$ 

We'll first rearrange this equation to make it easier to work with:

<a name="eq6"></a>

$$\mathsf{P( A \cap B) = P(A \lvert B)P(B)}\tag{6} $$ 

The probability of $$\mathsf{A}$$ and $$\mathsf{B}$$ both occurring, $$\mathsf{P( A \cap B)}$$, is identical to the probability of $$\mathsf{B}$$ and $$\mathsf{A}$$ both occurring, $$\mathsf{P( B \cap A)}$$.  Therefore, we can rewrite the [Equation 6](#eq6) as:

<a name="eq7"></a>

$$\mathsf{P( B \cap A) = P(A \lvert B)P(B)}\tag{7} $$ 
 

Switching the roles of $$\mathsf{A}$$ and $$\mathsf{B}$$ in [Equation 6](#eq6) provides another expression for  $$\mathsf{P( B \cap A)}$$:

<a name="eq8"></a>

$$\mathsf{P(B \cap A) = P( B \lvert A)P(A)}\tag{8} $$ 

We can see that the right hand side of [Equation 7](#eq7) and [Equation 8](#eq8) must be equal.  Therefore:

$$\mathsf{P(A \lvert B)P(B) = P( B \lvert A)P(A)}\tag{9}$$ 

Solving for $$\mathsf{P(A \lvert B)}$$, we arrive a **Bayes' Theorem**

$$\mathsf{P(A \lvert B) = \frac{P( B \lvert A)P(A)}{P(B)}}\tag{10}$$ 

In words, Bayes' Theorem simply another way of stating that the probability of $$\mathsf{A}$$ occuring given $$\mathsf{B}$$ has occured, $$\mathsf{P(A \lvert B)}$$, is equal to the probability of $$\mathsf{A}$$ and $$\mathsf{B}$$ both occuring, $$\mathsf{P( B \lvert A)P(A)}$$, divided by the probability of $$\mathsf{B}$$ occuring.  Notice that this is the exact same statement that we made when discussing conditional dependence at the end of the first section!  If you are having a hard time intuitively undestanding Bayes' Theorem, I encourage you to [go back](#goback) to the first section and convince yourself that Bayes' Theorem is simply a different mathematical statement for the easier-to-grasp conditional probability theorem.

Often, we need to use the **Law of Total Probability** to calculate the denominator, $$\mathsf{P(B)}$$, in Bayes' Theorem.  Therefore, it's sometimes more useful to combine the two statements into the more explicit form of Bayes' Theorem:

$$\mathsf{P(A \lvert B) = \frac{P( B \lvert A)P(A)}{\sum_{i=1}^n P(B \lvert A_i)P(A_i)}}\tag{11}$$ 

where $$\mathsf{A_i}$$ denotes the partitions of the sample space.

Returning to our sport's team analogy, suppose we know the following pieces of information:

* The probability of our favorite team winning a game, $$\mathsf{P(W)}$$
* The probability of our favorite team playing at home, $$\mathsf{P(H)}$$
* The probability of our favorite team winning a game given that they are playing at home, $$\mathsf{P(W \lvert H)}$$

With Bayes' theorem we can combine all of this information to answer the question "If we know that our team won a game, what is the probability that they were playing at home, $$\mathsf{P(H \lvert W)}$$?"

# <a name="naiveBayes"></a> Naive Bayes Classifiers

A Naive Bayes Classifier is a machine learning algorithm that uses Bayes' Theorem to predict the class that a sample belongs to, given a number of features that describe that sample.  If we collect information on $$\mathsf{n}$$ different features.  We can write the values of these features for a particular sample as a $$\mathsf{1 \times n}$$ vector $$\mathbf{X} = \{x_1,x_2,x_3,...,x_n\}$$ known as the **evidence**.

Our goal is to determine which class the sample belongs to.  This boils down to calculating the probability for each class given the evidence vector $$\mathbf{X}$$.  Bayes Theorem tells us that for a particular class, $$\mathsf{C_i}$$, this is given by

<a name="eq12"></a>

$$\mathsf{ P(C_i \lvert \mathbf{X}) = \frac{P( \mathbf{X} \lvert C_i)P(C_i)}{P(\mathbf{X})} }\tag{12}$$ 

To evaluate Bayes' theorem we need to determine the three probabilities on the right hand side of [Equation 12](#eq12).  To do so, we'll need to use a training set of data for which the features and class are known.  Then, using the data in our training set, we know:

<a name="eq13"></a>

$$\mathsf{ P(C_i)= \frac{number \; of \; training \; samples \; in \; class \; C_i}{number \; of \; training \; samples} \tag{13}}$$

In pratice, the probability of the observed features occuring given the sample belongs to class $$\mathsf{C_i}$$ can be hard to calculate.
To simplify the calculation, Naive Bayes Classifiers assumes that the features are **conditional independent**.  This means that once we know the sample belongs to a particular class, knowledge of the outcome of one feature does not grant us knowledge of the outcome of any other feature. This assumption is often naive in the real world, since separate features such as temperature and humidity are often correlated &mdash; hence the name *Naive* Bayes Classifier.  None the less, the algorithm often performs better than much more complex machine learning models.

Mathematically, the consquence of this assumption is that the probability of the observing the evidence $$\mathbf{X}$$, given that the sample belongs to some class $$\mathsf{C_i}$$, is equal to the product of the probabilities of each individual feature outcome given that the sample belongs to $$\mathsf{C_i}$$:

<a name="eq14"></a>


$$\mathsf{ P(\mathbf{X} \lvert C_i) = P(x_1 \lvert C_i)P(x_2 \lvert C_i) \dotsc P(x_n \lvert C_i) = \prod_{k=1}^n P(x_k \lvert C_i) \tag{14}}$$

The individual feature probabilities, given that the sample belongs to some class $$\mathsf{C_i}$$, can easily be determined from our training set

<a name="eq15"></a>


$$\mathsf{ P(x_k|C_i) = \frac{number \; training \; samples \; with \; class \; C_i \; and \; feature \; outcome \; x_k}{number \; of \; training \; samples \; with \; class \; C_i} \tag{15}}$$

The denominator in [Equation 12](#eq12), $$\mathsf{P(\mathbf{X})}$$, is the same regardless of which class we are looking at, so we don't actually need to calculate it. In fact, we shouldn't calculate it so that the algorithm will run faster.  That said, if we did want to calculate $$\mathsf{P(\mathbf{X})}$$ we would just use the **Law of Total Probability**.

Once we know $$P(C_i \lvert X)$$ for all classes, we can predict which class the sample belongs to based on a decision rule.  One of the most common and straightforward decision rule, known as the "maximum posteriori hypothesis," is to predict that the sample belongs to whichever class has the highest probability $$P(X \lvert C_i)$$.

As a final note, we may want to consider how to use a Naive Bayes Classifier when we have a feature that is continuous (such as the amount of rainfall on a game day) rather than discrete (such as whether or not it rained on a game day).  In such a case, we typically assume that the continous values have a Gaussian distribution with mean $$\mu$$ and standard deviation $$\sigma$$, such that

<a name="eq16"></a>

$$\mathsf{P(x_k|C_i) = \frac{1}{\sqrt{2\pi \sigma}} exp \left(- \frac{(x_k-\mu)^2}{2 \sigma^2} \right) \tag{16}}$$


# <a name="naiveEx"></a> Applying the Naive Bayes Classifier


We'll wrap up this discussion by applying the Naive Bayes' Classifier to our sports team analogy.  Suppose our favorite sports team played 100 games with a record of 65 wins - 35 losses.  During the season, we tracked the team's wins and losses, as well as information about three features &mdash; whether our team was playing at home, whether it was raining during the game, and who the experts thought would win the game.  The information produced the following frequency tables:



| Feature: Playing At Home            | Games Won  | Games Lost | Total Games
| :------------- |:-------------:|:------:| :----------:
| At Home      | 40 | 20 | **60**
| Away         | 25 | 15 | **40**
| **Total**        | **65** | **35** | **100**

| Feature: Weather During Game        | Games Won  | Games Lost | Total Games
| :------------- |:-------------:|:------:| :----------:
| Raining       | 20    | 10 | **30**
| Not Rain      | 45    | 25 | **70**
| **Total**        | **65** | **35** | **100**

| Feature: Expert's Predictions     | Games Won  | Games Lost | Total Games
| :------------- |:-------------:|:------:| :----------:
| Experts Predict a Win    | 45    |  10 | **55**
| Experts are split        | 10     |  5    | **15**
| Experts Predict a Loss   | 10    |  20   | **30**
| **Total**        | **65** | **35** | **100**


Tomorrow, our team is about to play their first play off game.  We'd like to predict whether they will win or not based on the features we have tracked all season.  We know that our team will be playing at home, that it will be raining on game day, and that the experts are split on who will win the game.  In this case, our evidence vector looks like this:

$$\mathbf{X} = \mathsf{\{At \; Home, Raining, Experts \; are \; split\}}$$

There are two possible outcomes to predict &mdash; our team winning (denoted with $$\mathsf{W}$$) and our team losing (denoted with $$\mathsf{L}$$).  Bayes' theorem tells us the probability of each of these outcomes, given the evidence vector for the upcoming game:

$$\mathsf{ P(W \lvert \mathbf{X}) = \frac{P( \mathbf{X} \lvert W)P(W)}{P(\mathbf{X})} }$$ 

$$\mathsf{ P(L \lvert \mathbf{X}) = \frac{P( \mathbf{X} \lvert L)P(L)}{P(\mathbf{X})} }$$ 



From the frequency tables above, and from [Equation 13](#eq13), we know that:

$$\mathsf{ P(W)= \frac{number \; of \; games \; \; Won}{Number \; of \; Games}} = \frac{65}{100} = 0.65 $$

$$\mathsf{ P(L)= \frac{number \; of \; games \; \; Lost}{Number \; of \; Games}} = \frac{35}{100} = 0.35$$

Assuming that our features are conditionally independent, [Equation 14](#eq14) and [Equation 15](#eq15) tell us the probability of observing the evidence  $$\mathbf{X}$$ given that our team wins.


$$\mathsf{ P(\mathbf{X} \lvert W) = P(At \; Home \lvert W) \; P(Raining \lvert W) \; P(Experts \; are \; Split \lvert W) }$$

$$\mathsf{ P(\mathbf{X} \lvert W) = \left( \frac{\; Wins \; at \; Home}{Games \; at \; Home} \right) 
\left( \frac{\; Wins \; when \; Raining}{Games \; when \; Raining} \right)
\left( \frac{Wins \; when \; Experts \; Split}{Games \; when \; Experts \; Split} \right) }$$

$$\mathsf{ P(\mathbf{X} \lvert W) = \left(\frac{40}{60}\right) \left(\frac{20}{30}\right) \left(\frac{10}{15}\right) = 0.30 }$$ 

Repeating the process above, given that our team loses rather than wins tells us:

$$\mathsf{ P(\mathbf{X} \lvert L) = \left(\frac{20}{60}\right) \left(\frac{10}{30}\right) \left(\frac{5}{15}\right) = 0.04 }$$ 

We don't need to calculate $$\mathsf{P(\mathbf{X})}$$ since the result does not depend on whether we win or lose, but for completeness I'll go ahead and do so using the Law of Total Probability, [Equation 3](#eq):

$$\mathsf{P(\mathbf{X}) = P(X \lvert W)P(W) +  P(X \lvert L)P(L)} = (0.3 \times 0.65) + (0.04 \times 0.35) = 0.21$$

Finally, we can plug all of these numbers into Bayes' Theorem and arrive at the following probabilities for winning or losing given the evidence vector for the upcoming game:

$$\mathsf{ P(W \lvert \mathbf{X}) = \frac{P( \mathbf{X} \lvert W)P(W)}{P(\mathbf{X})} = \frac{0.3 \times 0.65}{0.21} = 0.93 }$$ 

$$\mathsf{ P(L \lvert \mathbf{X}) = \frac{P( \mathbf{X} \lvert L)P(L)}{P(\mathbf{X})} = \frac{0.04 \times 0.35}{0.21} = 0.07 }$$ 

The Naive Bayes Classifier says our team has a 93% chance of winning the upcoming game, and only a 7% chance of losing!  We can safely predict that our team will win game! 


# Closing remarks

We've developed an understanding of Naive Bayes Classifiers and the Bayesian statistics from which they are derived.  In the next post we'll learn how to extract useful features from our Pokemon Go tweets, and use them to train a sentiment analyzer implemented with the Natural Language Tool Kit's (NLTK) built-in Naive Bayes Classifier.