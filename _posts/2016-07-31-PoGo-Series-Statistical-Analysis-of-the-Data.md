---
layout: post
title: PoGo Series&#58; Statistical Analysis of Data
comments: true
---

Measuring the dominance of each time while properly accounting for statistical and systematic errors in our data.


<!--more-->

Table of Contents:

1. [Introduction to the Series](#http://www.raknoche.github.io/2016/07/20/PoGo-Series-Intro/)
2. [The Twitter API](#http://www.raknoche.github.io/2016/07/21/PoGo-Series-TwitterAPI/)
3. [The Tweepy Library](#http://www.raknoche.github.io/2016/07/23/PoGo-Series-Tweepy/) 
4. [Naive Bayes Classifiers](#http://www.raknoche.github.io/2016/07/24/PoGo-Series-NaiveBayesClassifier/)
5. [Training a Sentiment Analyzer](#http://www.raknoche.github.io/2016/07/28/PoGo-Series-Sentiment-Analyzer/)
6. [Statistical Analysis of the Data](#http://www.raknoche.github.io/2016/07/31/PoGo-Series-Statistical-Analysis-of-the-Data/)
7. [Visualizing the Data with Choropleth Maps](#http://www.raknoche.github.io/2016/08/03/PoGo-Series-Making-a-Choropleth-Map/)

---------------------

Welcome to the second-to-last post in our Pokemon Go analysis series.  [Last time](#http://www.raknoche.github.io/2016/07/28/PoGo-Series-Sentiment-Analyzer/), we trained a sentiment analyzer that will allow us to remove negative tweets from our collection.  In this post, we'll use this analyzer to clean up our data and determine the dominance of each Pokemon Go team in each state.  Once we've cleaned the data, we'll count the number of tweets referencing each Pokemon Go team in every state.  In the end, we'll use the fraction of tweets referencing each team as a measure of the dominance of the teams in each state.  Along the way, we'll discuss how to properly account for statistical and systematic errors in our measurement.


We'll cover the following topics:

1. [Cleaning the data](#cleaning)
2. [Finding the location of each tweet](#location)
3. [Measuring team dominance in each state](#dominance)
4. [Statistical analysis of the data](#stats)


# <a name="cleaning"></a> Cleaning the Data

Before we begin, we'll need to load the our collection of tweets into Pandas.  We already did this in our [last post](#http://www.raknoche.github.io/2016/07/28/PoGo-Series-Sentiment-Analyzer/), and saved to the result to a csv file. We can load that csv file with the following code:


```python
#This is needed to load our csv
import pandas as pd

#We'll also use numpy to do some math later
import numpy as np
```


```python
#Load the full collection of PoGo tweets
PoGo_tweets = pd.read_csv('PoGo_Sentiment_AllTweets.csv')
PoGo_tweets.drop('Unnamed: 0', axis=1, inplace=True)
PoGo_tweets.head(n=5)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>screenName</th>
      <th>userId</th>
      <th>text</th>
      <th>location</th>
      <th>multi-team</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>desmond_ayala</td>
      <td>2.953472e+09</td>
      <td>Which pokemon go team did y'all chose? #valor</td>
      <td>Caldwell, ID</td>
      <td>False</td>
    </tr>
    <tr>
      <th>1</th>
      <td>aphrospice</td>
      <td>1.629086e+07</td>
      <td>#Magikarp practicing his struggle skills in th...</td>
      <td>Brooklyn, NY</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ABellgowan</td>
      <td>1.681036e+09</td>
      <td>Pokemon Go is taking over my life #TeamInstinct</td>
      <td>Bixby, OK</td>
      <td>False</td>
    </tr>
    <tr>
      <th>3</th>
      <td>JangoSnow</td>
      <td>1.057434e+07</td>
      <td>Go Team Instinct! I like underdogs. :)  https:...</td>
      <td>Los Angeles, CA</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4</th>
      <td>EmberLighta2</td>
      <td>7.513920e+17</td>
      <td>#TeamMystic has total control of Niagara Falls!!</td>
      <td>Niagara Falls, NY</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>



Recall that the csv file has already had all tweets that reference multiple teams removed, since we decided it would be too hard to identify which team the Twitter user was on from such tweets. 

We'd like to remove any negatively-toned tweets from our collection so that we can assume any remaining tweets about the team indicates that the Twitter user is a member of that team.  We can do so by applying the sentiment analyzer that we developed in our [last post](#http://www.raknoche.github.io/2016/07/28/PoGo-Series-Sentiment-Analyzer/).  First, we use `pickle` to load the sentiment analyzer and feature list that we created.


```python
#load the classifier
import pickle
f = open('PoGo_tweet_classifier.pickle', 'rb')
classifier = pickle.load(f)
f.close()

#load the list of features used in the classifier
with open('PoGo_classifier_feats.pickle', 'rb') as f:
    word_features = pickle.load(f)
```

We'll also need to use the `filter_tweets` and `extract_features` functions that we wrote for our sentiment analyzer.  In this case, I've slightly modified the `filter_tweets` function &mdash; it now produces a list of filtered unigrams and bigrams for each tweet instead of a tuple, since we have no manually labeled sentiment to pair with such a list. 


```python
#Load the required modules
import nltk
from nltk.metrics import BigramAssocMeasures
from nltk.collocations import BigramCollocationFinder
import itertools
import string


#Set of punctuation to exclude from unigrams and bigrams
exclude = set(string.punctuation)

#List of team-identifying words to exclude from unigrams and bigrams
excluded_words = ['teammystic','mystic','teamblue','blue',\
                  'teaminstinct','instinct','teamyellow','yellow',\
                  'teamvalor','valor','teamred','red']


#Function that provides a list of filtered unigrams and bigrams from each tweet
def filter_tweets(tweet_text):
    words_filtered=[]

    #For each word in the tweet, filter on our feature requirements.
    for word in tweet_text.split(): 

        #Remove punctuation
        word = ''.join(ch for ch in word if ch not in exclude)

        #Remove one letter words
        if len(word) >= 1: 

                #treat URLs the same
                if word[:4] == 'http':
                    word='http'

                #remove hashtags
                if word[0] == '#': 
                    word=word[1:]

                #remove team identifiers
                if (word.lower() not in excluded_words):

                    #require lower case
                    words_filtered.append(word.lower()) 
    
    #If the word list contains only duplicates of one word, it causes problems for bigram finder
    #In this case, don't bother trying to find bigrams, just find the unigram since there are no bigrams anyway
    if len(set(words_filtered)) == 1:
        tweet_feats = words_filtered[0]
    else:  
        #Identify top 200 bigams in the filtered word list using chi_sq measure of importance
        bigram_finder = BigramCollocationFinder.from_words(words_filtered)
        bigrams = bigram_finder.nbest(BigramAssocMeasures.chi_sq, 200)  

        tweet_feats = [ngram for ngram in itertools.chain(words_filtered, bigrams)]

    return tweet_feats


#Feature extractor - determines which word features are in each tweet
def extract_features(filtered_tweet):

    #list of unigrams and bigrams in the tweet
    filtered_tweet_words = set(filtered_tweet)
    
    #Define a features dictionary
    features = {}

    #Loop of all word features
    for word in word_features:
        
        #Set 'contains(word_feature)' as a key in the dictionary
        #Set the value for that key to True or False
        features['contains(%s)' % str(word)] = (word in filtered_tweet_words)

    #Return the final features dictionary for that tweet
    return features
```

To classify the sentiment of each tweets in our collection, we first apply the `filter_tweets` function to extract the filtered unigrams and bigrams from the tweet.  We use that list of unigrams and bigrams to evaluate the `contains(word_feature)` statements which we used as features for our sentiment analyzer.  Finally, we pass the feature values to our sentiment analyzer and place the result in a new "sentiment" column in our data frame.


```python
#Classifying tweets
def ClassifyTweets(dFrame):
    for row in range(len(dFrame)):
        
        #Get filtered unigrams and bigrams
        filtered_text = filter_tweets(dFrame.ix[row,'text'])
        
        #Evaluate the contains(word) statements
        tweet_feats = extract_features(filtered_text)
        
        #Add result to new sentimenet column in the dataframe
        dFrame.ix[row,'sentiment']=classifier.classify(tweet_feats)   
```


```python
ClassifyTweets(PoGo_tweets)
PoGo_tweets.head(n=5)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>screenName</th>
      <th>userId</th>
      <th>text</th>
      <th>location</th>
      <th>multi-team</th>
      <th>sentiment</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>desmond_ayala</td>
      <td>2.953472e+09</td>
      <td>Which pokemon go team did y'all chose? #valor</td>
      <td>Caldwell, ID</td>
      <td>False</td>
      <td>positive</td>
    </tr>
    <tr>
      <th>1</th>
      <td>aphrospice</td>
      <td>1.629086e+07</td>
      <td>#Magikarp practicing his struggle skills in th...</td>
      <td>Brooklyn, NY</td>
      <td>False</td>
      <td>positive</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ABellgowan</td>
      <td>1.681036e+09</td>
      <td>Pokemon Go is taking over my life #TeamInstinct</td>
      <td>Bixby, OK</td>
      <td>False</td>
      <td>positive</td>
    </tr>
    <tr>
      <th>3</th>
      <td>JangoSnow</td>
      <td>1.057434e+07</td>
      <td>Go Team Instinct! I like underdogs. :)  https:...</td>
      <td>Los Angeles, CA</td>
      <td>False</td>
      <td>positive</td>
    </tr>
    <tr>
      <th>4</th>
      <td>EmberLighta2</td>
      <td>7.513920e+17</td>
      <td>#TeamMystic has total control of Niagara Falls!!</td>
      <td>Niagara Falls, NY</td>
      <td>False</td>
      <td>positive</td>
    </tr>
  </tbody>
</table>
</div>



If a state in the US has a lot of Twitter users we'll have plenty of tweets for our team dominance measurement.  In this case, removing tweets that our sentiment analyzer has identified as negative will reduce the total error on our measurement.  However, if a particular state has very few tweets about each Pokemon Go team, our sentiment analyzer will reduce our already small sample size.  In turn, this will increase the total error on our team dominance measurement.  I'll explain why this is the case in the upcoming [statistical analysis](#stats) sections, but for now it is sufficient to know that we will not always want to remove the negative tweets from our data set.  Therefore, we must create two branches of our data frame &mdash; one in which we do remove negative tweets, and one in which we do not.


```python
#Creating a new data frame where we do not apply sentiment analysis
PoGo_tweets_nosenti = PoGo_tweets

#Applying the positive sentiment restriction on the original dataframe
PoGo_tweets = PoGo_tweets[PoGo_tweets['sentiment'] == 'positive']
PoGo_tweets = PoGo_tweets.reset_index(drop=True)
```

Next, we'll want to require that each tweet is coming from a unique user.  If there happens to be a Twitter addict who Tweets about their Pokemon Go team every hour, we don't want to count them as a member of that team multiple times.  We can ensure this doesn't happen by indicating the first tweet from each user in a new column of our data set.


```python
def FindUniqueUsers(dFrame):
    unique_users=[]
    
    #Loop through the tweets in our dataframe
    for row in range(len(dFrame)):
        
        #If we have already seen this Twitter user, flag the tweet as a repeat
        if dFrame.ix[row,'screenName'] in unique_users:
            dFrame.ix[row,'repeatUser'] = True
            
        #Otherwise, add the user to a list so we can identify if they tweet again 
        else:
            dFrame.ix[row,'repeatUser'] = False
            unique_users.append(dFrame.ix[row,'screenName'])
```


```python
#Apply the unique user function to both of our data frames
FindUniqueUsers(PoGo_tweets_nosenti)
FindUniqueUsers(PoGo_tweets)
```


```python
#Remove all but the first tweet from each user in our data frames
PoGo_tweets = PoGo_tweets[PoGo_tweets['repeatUser'] == False]
PoGo_tweets_nosenti = PoGo_tweets_nosenti[PoGo_tweets_nosenti['repeatUser'] == False]

#Reindex the data frames
PoGo_tweets = PoGo_tweets.reset_index(drop=True)
PoGo_tweets_nosenti = PoGo_tweets_nosenti.reset_index(drop=True)
```

<h1> <a name="location"></a> Finding the location of each tweet </h1>

Now that we have removed multi-team tweets, negatively-toned tweets, and repeated tweets from the same user from our collection, we are ready to count the number of tweets about each Pokemon Go team in each state.  First, we need to determine which state each tweet came from.  The location column of our data frame almost does this for us, but as you can see below, some tweets have location information in the format `State, Country`, while others have location information in the format `City, State Abbreviation`. 


```python
PoGo_tweets.tail(n=3)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>screenName</th>
      <th>userId</th>
      <th>text</th>
      <th>location</th>
      <th>multi-team</th>
      <th>sentiment</th>
      <th>repeatUser</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>9434</th>
      <td>TheJakeTyler</td>
      <td>42497434.0</td>
      <td>#TeamMystic let's get it!!!!</td>
      <td>Virginia, USA</td>
      <td>False</td>
      <td>positive</td>
      <td>False</td>
    </tr>
    <tr>
      <th>9435</th>
      <td>LivAboveAverage</td>
      <td>333182239.0</td>
      <td>@AnthonyChott GO TEAM MYSTIC!!! #TeamMystic</td>
      <td>Chesterfield, MO</td>
      <td>False</td>
      <td>positive</td>
      <td>False</td>
    </tr>
    <tr>
      <th>9436</th>
      <td>TylerDurden919</td>
      <td>334394891.0</td>
      <td>10:45 Saturday morning, everyone's at @Bojangl...</td>
      <td>Durham, NC</td>
      <td>False</td>
      <td>positive</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>



We'd like to extract the state information, regardless of which location format is being used.  To do so, we first define a dictionary of state abbreviations and the corresponding state name.  We also will want to create stand-alone lists of state abbreviations and state names from this dictionary.


```python
#Dictionary of state abbreviations and names
states = {
        'AK': 'Alaska',
        'AL': 'Alabama',
        'AR': 'Arkansas',
        'AZ': 'Arizona',
        'CA': 'California',
        'CO': 'Colorado',
        'CT': 'Connecticut',
        'DC': 'District of Columbia',
        'DE': 'Delaware',
        'FL': 'Florida',
        'GA': 'Georgia',
        'HI': 'Hawaii',
        'IA': 'Iowa',
        'ID': 'Idaho',
        'IL': 'Illinois',
        'IN': 'Indiana',
        'KS': 'Kansas',
        'KY': 'Kentucky',
        'LA': 'Louisiana',
        'MA': 'Massachusetts',
        'MD': 'Maryland',
        'ME': 'Maine',
        'MI': 'Michigan',
        'MN': 'Minnesota',
        'MO': 'Missouri',
        'MS': 'Mississippi',
        'MT': 'Montana',
        'NC': 'North Carolina',
        'ND': 'North Dakota',
        'NE': 'Nebraska',
        'NH': 'New Hampshire',
        'NJ': 'New Jersey',
        'NM': 'New Mexico',
        'NV': 'Nevada',
        'NY': 'New York',
        'OH': 'Ohio',
        'OK': 'Oklahoma',
        'OR': 'Oregon',
        'PA': 'Pennsylvania',
        'PR': 'Puerto Rico',
        'RI': 'Rhode Island',
        'SC': 'South Carolina',
        'SD': 'South Dakota',
        'TN': 'Tennessee',
        'TX': 'Texas',
        'UT': 'Utah',
        'VA': 'Virginia',
        'VT': 'Vermont',
        'WA': 'Washington',
        'WI': 'Wisconsin',
        'WV': 'West Virginia',
        'WY': 'Wyoming'
}

#Create a list of state abbreviations
state_abbrevs=[state for state in states] 

#Create a list of state names
state_names=[states[state] for state in states] 
```

For each tweet in our data frame, we can scan the location information to look for the state abbreviations or state names from our dictionary.  Some location information, such as "New York, NY" or "Alabama, New York" (yes, Alabama is a real city in New York) will contain multiple state names or abbreviations.  In this case, we'll want to identify the second name or abbreviation that appears in the location information.  The one exception to this rule is users from "District of Columbia, Washington," where we'll want to use the first "state" name (District of Columbia) that appears.  Once we've extracted the state from the location information, we'll add it to a new column in our data frame.


```python
#Function to determine the state that each tweet originated in
def GetTweetState(dFrame):
    
    #Get a list of all the tweet's locations from our dataframe
    loc_list=list(dFrame['location'])
    
    state_list=[]
    
    #Loop over the location info for each tweet
    for info in loc_list:
        
        #See if a state abbreviation or name is in the location information
        new_state = [state for state in state_names if state in info] + \
                    [states[state] for state in states if state in info]

        #Handle cases like "New York, NY" and "Alabama, New York"
        if (len(new_state) > 1) and new_state[0] != 'District of Columbia':
            new_state = new_state[1].split('junk')
            
        #Handle "District of Columbia, Washington"     
        elif len(new_state) > 1 and new_state[0] == 'District of Columbia':
            new_state = new_state[0].split('junk')
            
        #Handle cases where no state is mentioned    
        if not new_state:
            new_state = ['None']

        #After we determine the state, add it to a list
        state_list+=new_state
     
    #Once we have the entire list of states, add it to the dataframe
    dFrame['state']=state_list
```


```python
GetTweetState(PoGo_tweets)
GetTweetState(PoGo_tweets_nosenti)

PoGo_tweets.tail(n=3)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>screenName</th>
      <th>userId</th>
      <th>text</th>
      <th>location</th>
      <th>multi-team</th>
      <th>sentiment</th>
      <th>repeatUser</th>
      <th>state</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>9434</th>
      <td>TheJakeTyler</td>
      <td>42497434.0</td>
      <td>#TeamMystic let's get it!!!!</td>
      <td>Virginia, USA</td>
      <td>False</td>
      <td>positive</td>
      <td>False</td>
      <td>Virginia</td>
    </tr>
    <tr>
      <th>9435</th>
      <td>LivAboveAverage</td>
      <td>333182239.0</td>
      <td>@AnthonyChott GO TEAM MYSTIC!!! #TeamMystic</td>
      <td>Chesterfield, MO</td>
      <td>False</td>
      <td>positive</td>
      <td>False</td>
      <td>Missouri</td>
    </tr>
    <tr>
      <th>9436</th>
      <td>TylerDurden919</td>
      <td>334394891.0</td>
      <td>10:45 Saturday morning, everyone's at @Bojangl...</td>
      <td>Durham, NC</td>
      <td>False</td>
      <td>positive</td>
      <td>False</td>
      <td>North Carolina</td>
    </tr>
  </tbody>
</table>
</div>



# <a name="dominance"></a> Measuring team dominance in each state

Now that we've identified the state each tweet is from, we can count how many tweets about each Pokemon Go team we collected in each state.  First, we initialize a new data frame with a row for each state.


```python
#stateInfo and stateInfo_ns
def MakeStateInfo(dFrame):
    dFrame['State']=states.values()
    dFrame['Num Tweets'] = 0
    dFrame['Num Red'] = 0
    dFrame['Num Blue'] = 0
    dFrame['Num Yellow'] = 0
```


```python
stateInfo = pd.DataFrame()
stateInfo_nosenti = pd.DataFrame()

MakeStateInfo(stateInfo)
MakeStateInfo(stateInfo_nosenti)

stateInfo.head(n=5)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>State</th>
      <th>Num Tweets</th>
      <th>Num Red</th>
      <th>Num Blue</th>
      <th>Num Yellow</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Pennsylvania</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Delaware</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Nebraska</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Michigan</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Idaho</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



The `stateInfo` dataframe is indexed with numbers by default.  Since each row represents a state, it would be more convenient to use the state names as the index.  We can tell Pandas to do this using the `set_index` method.


```python
stateInfo = stateInfo.set_index(['State'])
stateInfo_nosenti = stateInfo_nosenti.set_index(['State'])

stateInfo.head(n=5)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Num Tweets</th>
      <th>Num Red</th>
      <th>Num Blue</th>
      <th>Num Yellow</th>
    </tr>
    <tr>
      <th>State</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Pennsylvania</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Delaware</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Nebraska</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Michigan</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Idaho</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



Now, we simply loop over each tweet in our collection.  For each tweet, we search for team-identifying phrases.  Once we've identified the team the tweet is referring to, we increase the count for that team in the corresponding state by one.


```python
#Lists of team identifying phrases
yellow_text = ['#teaminstinct','#teamyellow','#instinct','team instinct']
blue_text = ['#teammystic','#teamblue','#mystic','team mystic']
red_text = ['#teamvalor','#teamred','#valor','team valor']

#Function to populate the stateInfo data frame
def PopStateInfo(state_dFrame,tweet_dFrame):
    
    #Loop over the tweet collection
    for idx in range(len(tweet_dFrame)):
        
        #If we have state information for the tweet
        if tweet_dFrame.ix[idx,'state'] != 'None':
            
            #Search for the team identifying phrases and
            #Add one count to that team in state where the tweet came from 
            if any(hashtag in tweet_dFrame.ix[idx,'text'].lower() for hashtag in yellow_text):
                state_dFrame.ix[tweet_dFrame.ix[idx,'state'],'Num Yellow'] += 1
            elif any(hashtag in tweet_dFrame.ix[idx,'text'].lower() for hashtag in blue_text):
                state_dFrame.ix[tweet_dFrame.ix[idx,'state'],'Num Blue'] += 1
            elif any(hashtag in tweet_dFrame.ix[idx,'text'].lower() for hashtag in red_text):
                state_dFrame.ix[tweet_dFrame.ix[idx,'state'],'Num Red'] += 1

    #After we've process all tweets
    #Count all the tweets in each state by adding red + blue + yellow
    state_dFrame['Num Tweets'] = state_dFrame.sum(axis=1)
```


```python
PopStateInfo(stateInfo,PoGo_tweets)
PopStateInfo(stateInfo_nosenti,PoGo_tweets_nosenti)

stateInfo.head(n=5)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Num Tweets</th>
      <th>Num Red</th>
      <th>Num Blue</th>
      <th>Num Yellow</th>
    </tr>
    <tr>
      <th>State</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Pennsylvania</th>
      <td>275</td>
      <td>111</td>
      <td>97</td>
      <td>67</td>
    </tr>
    <tr>
      <th>Delaware</th>
      <td>27</td>
      <td>13</td>
      <td>7</td>
      <td>7</td>
    </tr>
    <tr>
      <th>Nebraska</th>
      <td>64</td>
      <td>26</td>
      <td>23</td>
      <td>15</td>
    </tr>
    <tr>
      <th>Michigan</th>
      <td>324</td>
      <td>123</td>
      <td>115</td>
      <td>86</td>
    </tr>
    <tr>
      <th>Idaho</th>
      <td>27</td>
      <td>11</td>
      <td>9</td>
      <td>7</td>
    </tr>
  </tbody>
</table>
</div>



Our metric for the dominance of each team in a state will simply be the fraction of tweets from that state that reference a particular team.  We can easily add this fraction to our data frame


```python
#Function to populate the stateInfo data frame
def CalcDominance(dFrame):
    dFrame['R Frac']=dFrame['Num Red']/dFrame['Num Tweets']
    dFrame['Y Frac']=dFrame['Num Yellow']/dFrame['Num Tweets']
    dFrame['B Frac']=dFrame['Num Blue']/dFrame['Num Tweets']
```


```python
CalcDominance(stateInfo)
CalcDominance(stateInfo_nosenti)

stateInfo.head(n=5)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Num Tweets</th>
      <th>Num Red</th>
      <th>Num Blue</th>
      <th>Num Yellow</th>
      <th>R Frac</th>
      <th>Y Frac</th>
      <th>B Frac</th>
    </tr>
    <tr>
      <th>State</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Pennsylvania</th>
      <td>275</td>
      <td>111</td>
      <td>97</td>
      <td>67</td>
      <td>0.403636</td>
      <td>0.243636</td>
      <td>0.352727</td>
    </tr>
    <tr>
      <th>Delaware</th>
      <td>27</td>
      <td>13</td>
      <td>7</td>
      <td>7</td>
      <td>0.481481</td>
      <td>0.259259</td>
      <td>0.259259</td>
    </tr>
    <tr>
      <th>Nebraska</th>
      <td>64</td>
      <td>26</td>
      <td>23</td>
      <td>15</td>
      <td>0.406250</td>
      <td>0.234375</td>
      <td>0.359375</td>
    </tr>
    <tr>
      <th>Michigan</th>
      <td>324</td>
      <td>123</td>
      <td>115</td>
      <td>86</td>
      <td>0.379630</td>
      <td>0.265432</td>
      <td>0.354938</td>
    </tr>
    <tr>
      <th>Idaho</th>
      <td>27</td>
      <td>11</td>
      <td>9</td>
      <td>7</td>
      <td>0.407407</td>
      <td>0.259259</td>
      <td>0.333333</td>
    </tr>
  </tbody>
</table>
</div>



# <a name="stats"></a> Statistical Analysis of the data

In this section, I'll discuss how to properly account for statistical and systematic errors in each of our tweet counts, and how to propagate those errors to our team dominance metric.  Along the way, I'll also determine how many tweets are needed for our sentiment analyzer to improve the total error of our measurement.  I'll provide hyperlinks that will detail the mathematical concepts we'll be using, but without a strong understanding of statistics and calculus this section may be hard to follow.

## Counting statistics

In statistics, a [Poisson distribution](https://www.pp.rhul.ac.uk/~cowan/stat/notes/PoissonNote.pdf) is used to describe the number of times an event occurs within a time interval.  This representation is appropriate if:

* The number of times the event occurs ($$\mathsf{N}$$) in an interval is discrete (0,1,2,...)
* The event occurs independent of other events
* The rate at which events occur is approximately constant.
* The probability of events occurring at the same time is negligible.
* The probability of an event occurring in an interval is proportional to the length of the interval.

The number of Pokemon Go tweets within each state meets all of the above conditions, so we can use the Poisson distribution to determine the statistical error on our measurement.  The standard deviation of a Poisson distribution is given by the square root of the number of events observed.  (A derivation of this can be found in the previous link.)  Therefore, the statistical error of the count of tweets ($$\mathsf{N}$$) is simply:

$$\mathsf{ \sigma_{N,stat} = \sqrt{N} }\tag{1}$$

From our [last post](#http://www.raknoche.github.io/2016/07/28/PoGo-Series-Sentiment-Analyzer/), we know that 8.92% of our tweet collection is contaminated with negative tweets when we do not apply our sentiment analyzer.  Once we apply our sentiment analyzer to remove negative tweets, this contamination falls to 2.4%.  We can treat the contamination of negative tweets as a systematic uncertainty on the number of tweets we count in each state:

$$\mathsf{ \sigma_{N, sys} = \begin{cases}
0.0892*N_{without \; analyzer}  & \text{if analyzer is not applied} \\
0.0240*N_{with \; analyzer} & \text{if analyzer is applied}
\end{cases} }\tag{2}
$$

Since the statistical and systematic errors on our tweet counts are uncorrelated, we can [add them in quadrature](http://physics.stackexchange.com/questions/23441/how-to-combine-measurement-error-with-statistic-error) to find the total error on our tweet counts:

$$\mathsf{ \sigma_{N,tot} = \sqrt{ \sigma_{N,stat}^2 + \sigma_{N,sys}^2} = \begin{cases}
\sqrt{N_{without \; analyzer} + (0.0892*N_{without \; analyzer})^2}  & \text{if analyzer is not applied} \\
\sqrt{N_{with \; analyzer}+ (0.0240*N_{with \; analyzer})^2} & \text{if analyzer is applied}
\end{cases} }\tag{3}$$

## When is our sentiment analysis useful?

Now that we know how to calculate the total error on our tweet counts, we can determine how many tweets we need for our sentiment analyzer to be useful.  Applying our sentiment analyzer will lower the total number of tweets in our collection.  In turn, this will lower both the absolute statistical and systematic error of our measurement.  It's important to realize that even though the absolute error on our tweet counts decreases as the total number of tweets decreases, the relative size of the error when compared to the number of tweets can still increase.  Instead, we must evaluate when our sentiment analyzer will lower the [relative error](http://mathworld.wolfram.com/RelativeError.html) of our measurement.  That is, we need to determine when:

<a name="eq4"></a>
$$\mathsf{ \frac{ \sqrt{ N_{without \; analyzer} + 0.0892^2 * N_{without \; analyzer}^2 } } {N_{without \; analyzer}} \ge 
\frac{ \sqrt{ N_{with \; analyzer} + 0.0240^2 * N_{with \; analyzer}^2 } }{N_{with \; analyzer}} }\tag{4}$$

When we use our sentiment analyzer to reduce the number of negative tweets in our collection, any misclassified positive tweets will also be removed, and any misclassified negative tweets will remain.  Therefore, the number of remaining tweets after applying our sentiment analyzer is equal to:

<a name="eq5"></a>
$$\mathsf{ N_{with \; analyzer} = 0.0892*N_{without \;analyzer}*(1-R^-) + (1-0.0892)*N_{without \; analyzer}*R^+ = 0.732 * N_{without \;analyzer} }\tag{5}$$

Where I've used the fact that the negative recall of our classifier was $$\mathsf{R^- = 0.789}$$, and the positive recall of our classifier was $$\mathsf{R^+=0.783}$$. 

Plugging [Equation 5](#eq5) into [Equation 4](#eq4) and simplifying yields the result that our classifier will improve the relative error on our measurement when 

$$\mathsf{ N_{without \; analyzer} \ge 50 }\tag{6}$$


## Propagating errors to our team dominance metric

To make our team dominance map, we want to determine the fraction of tweets from each state that reference a particular team.  With the number of tweets about a team in a particular state given by $$\mathsf{N_{team}}$$, and the number of total tweets in that state given by $$\mathsf{N_{state}}$$, the dominance of a particular team in a state is given by:

$$\mathsf{T \equiv Team \; Dominance =  \frac{N_{team}}{N_{state}} } \tag{6}$$

To determine the error on the team dominance measurement, we need to propagate the individual errors from the numerator and denominator.  The general equation for propagating the errors of numerous variables, $$\mathsf{\mathbf{X} = \lbrace x_1,x_2,x_3,\dotsc,x_n \rbrace}$$, to some function of those variables $$\mathsf{F(\mathbf{X})}$$ is given by:

<a name="eq7"></a>
$$\mathsf{ \sigma_F ^2 = \sum_{i=1}^n \sigma_{x_i}^2 \left( \frac{ \delta F }{\delta x_i} \right)^2 + \sum_{i=1}^n \sum_{i \ne j}^n \sigma_{x_i,x_j}^2 \left( \frac{ \delta F }{\delta x_i} \right)\left( \frac{ \delta F }{\delta x_j} \right) }\tag{7}$$

where $$\mathsf{\sigma_{x_i,x_j}^2}$$ is the covariance of the variables $$\mathsf{x_i}$$ and $$\mathsf{x_j}$$.  In the case of uncorrelated variables, $$\mathsf{\sigma_{x_i,x_j}^2 = 0}$$, so the last term in [Equation 7](#eq7) vanishes.  

As it stands, the variables in our team dominance measurement, $$\mathsf{N_{team}}$$ and $$\mathsf{N_{state}}$$, are correlated, since $$\mathsf{N_{state}}$$ is equal to the sum of the tweet counts for each individual team.  To simplify our analysis, we can rewrite the team dominance equation in terms of the number of tweets referencing that team ($$\mathsf{N_{team}}$$) and the number of tweets referencing a different team ($$\mathsf{N_{other}}$$):

<a name="eq8"></a>
$$\mathsf{T =  \frac{N_{team}}{N_{team} + N_{other}} } \tag{8}$$

[Equation 8](#eq8) has no correlated variables, so we don't need to worry about measuring the covariance of any terms when propagating error measurements.  Therefore, with the team dominance denoted as $$\mathsf{T}$$,


$$ \mathsf{ \sigma_T ^2 = \sigma_{N_{team},tot}^2 \left( \frac{ \delta T}{\delta N_{team}} \right)^2 + \sigma_{N_{other},tot}^2 \left( \frac{ \delta T }{\delta N_{other}} \right)^2 } \tag{9}$$

We can use the [product rule](https://www.math.ucdavis.edu/~kouba/CalcOneDIRECTORY/productruledirectory/ProductRule.html) to calculate the partial derivative of $$\mathsf{F}$$ with respect to $$\mathsf{N_{team}}$$:

$$\mathsf{ \frac {\delta T}{\delta N_{team}} = \frac{N_{other}}{(N_{team} + N_{other})^2} } \tag{10}$$

Similarly, we can use the [chain rule](https://en.wikipedia.org/wiki/Chain_rule) to calculate the partial derivative of $$\mathsf{F}$$ with respect to $$\mathsf{N_{other}}$$:

$$\mathsf{ \frac {\delta T}{\delta N_{other}} = \frac{-N_{team}}{(N_{team} + N_{other})^2} } \tag{11}$$

## Statistical Analysis Code

Below, I've included the code that implements our statistical analysis.


```python
def PopErrInfo(dFrame,sys_err):
    
    #Systematic errors from negative tweet contamination
    y_sys_err = sys_err*dFrame['Num Yellow']
    r_sys_err = sys_err*dFrame['Num Red']
    b_sys_err = sys_err*dFrame['Num Blue']

    #Statistical errors from Poisson counting
    y_stat_err = dFrame['Num Yellow'] ** (1/2)
    r_stat_err = dFrame['Num Red'] ** (1/2)
    b_stat_err = dFrame['Num Blue'] ** (1/2)

    #Total errors
    y_tot_err = (y_stat_err ** 2 + y_sys_err **2) ** (1/2)
    r_tot_err = (r_stat_err ** 2 + r_sys_err **2) ** (1/2)
    b_tot_err = (b_stat_err ** 2 + b_sys_err **2) ** (1/2)


    #Propagation of total errors to the team dominance metric
    dFrame['R Frac Err']= ( (r_tot_err ** 2)*(dFrame['Num Blue'] + dFrame['Num Yellow'])**2 + \
                                dFrame['Num Red']**2 * (b_tot_err**2 + y_tot_err**2) ) ** (1/2) / \
                             (dFrame['Num Yellow'] + dFrame['Num Blue'] + dFrame['Num Red'])**2

    dFrame['Y Frac Err']= ( (y_tot_err ** 2)*(dFrame['Num Blue'] + dFrame['Num Red'])**2 + \
                                dFrame['Num Yellow']**2 * (b_tot_err**2 + r_tot_err**2) ) ** (1/2) / \
                             (dFrame['Num Yellow'] + dFrame['Num Blue'] + dFrame['Num Red'])**2


    dFrame['B Frac Err']= ( (b_tot_err ** 2)*(dFrame['Num Yellow'] + dFrame['Num Red'])**2 + \
                                dFrame['Num Blue']**2 * (y_tot_err**2 + r_tot_err**2) ) ** (1/2) / \
                             (dFrame['Num Yellow'] + dFrame['Num Blue'] + dFrame['Num Red'])**2

```


```python
sys_err_nosenti= 0.0892
sys_err_senti  = 0.0240

PopErrInfo(stateInfo,sys_err_senti)
PopErrInfo(stateInfo_nosenti,sys_err_nosenti)
```


```python
stateInfo.head(n=5)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Num Tweets</th>
      <th>Num Red</th>
      <th>Num Blue</th>
      <th>Num Yellow</th>
      <th>R Frac</th>
      <th>Y Frac</th>
      <th>B Frac</th>
      <th>R Frac Err</th>
      <th>Y Frac Err</th>
      <th>B Frac Err</th>
    </tr>
    <tr>
      <th>State</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Pennsylvania</th>
      <td>275</td>
      <td>111</td>
      <td>97</td>
      <td>67</td>
      <td>0.403636</td>
      <td>0.243636</td>
      <td>0.352727</td>
      <td>0.030429</td>
      <td>0.026448</td>
      <td>0.029600</td>
    </tr>
    <tr>
      <th>Delaware</th>
      <td>27</td>
      <td>13</td>
      <td>7</td>
      <td>7</td>
      <td>0.481481</td>
      <td>0.259259</td>
      <td>0.259259</td>
      <td>0.096439</td>
      <td>0.084531</td>
      <td>0.084531</td>
    </tr>
    <tr>
      <th>Nebraska</th>
      <td>64</td>
      <td>26</td>
      <td>23</td>
      <td>15</td>
      <td>0.406250</td>
      <td>0.234375</td>
      <td>0.359375</td>
      <td>0.061806</td>
      <td>0.053213</td>
      <td>0.060367</td>
    </tr>
    <tr>
      <th>Michigan</th>
      <td>324</td>
      <td>123</td>
      <td>115</td>
      <td>86</td>
      <td>0.379630</td>
      <td>0.265432</td>
      <td>0.354938</td>
      <td>0.027841</td>
      <td>0.025192</td>
      <td>0.027430</td>
    </tr>
    <tr>
      <th>Idaho</th>
      <td>27</td>
      <td>11</td>
      <td>9</td>
      <td>7</td>
      <td>0.407407</td>
      <td>0.259259</td>
      <td>0.333333</td>
      <td>0.094828</td>
      <td>0.084526</td>
      <td>0.090961</td>
    </tr>
  </tbody>
</table>
</div>



Note that in a given state the number of tweets from one team may exceed 50 counts, while the number of tweets from a different team may not.  In this case, the sentiment analyzer would improve the error on our measurement for one team, but not for the other.  To resolve this ambiguity, I calculated the average relative error for each of our team dominance metrics.  I then added a new column to our data frames that indicated if the sentiment analysis improved the average relative error for each state.


```python
#relative error = frac_err/frac
def AverageRelErr(dFrame):
    row_errs = []
    for row in range(len(dFrame)): 
        rel_errs = []
        if dFrame.ix[row,'Y Frac'] != 0:
            rel_errs.append(dFrame.ix[row,'Y Frac Err']/dFrame.ix[row,'Y Frac'])

        if dFrame.ix[row,'R Frac'] != 0:
            rel_errs.append(dFrame.ix[row,'R Frac Err']/dFrame.ix[row,'R Frac'])

        if dFrame.ix[row,'B Frac'] != 0:
            rel_errs.append(dFrame.ix[row,'B Frac Err']/dFrame.ix[row,'B Frac'])

        row_errs.append(np.average(rel_errs))
    return row_errs
```


```python
stateInfo['Average Rel Err']=AverageRelErr(stateInfo)
stateInfo_nosenti['Average Rel Err']=AverageRelErr(stateInfo_nosenti)
```


```python
stateInfo['Improved'] = stateInfo['Average Rel Err'] <= stateInfo_nosenti['Average Rel Err']
stateInfo.head(n=5)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Num Tweets</th>
      <th>Num Red</th>
      <th>Num Blue</th>
      <th>Num Yellow</th>
      <th>R Frac</th>
      <th>Y Frac</th>
      <th>B Frac</th>
      <th>R Frac Err</th>
      <th>Y Frac Err</th>
      <th>B Frac Err</th>
      <th>Average Rel Err</th>
      <th>Improved</th>
    </tr>
    <tr>
      <th>State</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Pennsylvania</th>
      <td>275</td>
      <td>111</td>
      <td>97</td>
      <td>67</td>
      <td>0.403636</td>
      <td>0.243636</td>
      <td>0.352727</td>
      <td>0.030429</td>
      <td>0.026448</td>
      <td>0.029600</td>
      <td>0.089287</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Delaware</th>
      <td>27</td>
      <td>13</td>
      <td>7</td>
      <td>7</td>
      <td>0.481481</td>
      <td>0.259259</td>
      <td>0.259259</td>
      <td>0.096439</td>
      <td>0.084531</td>
      <td>0.084531</td>
      <td>0.284132</td>
      <td>False</td>
    </tr>
    <tr>
      <th>Nebraska</th>
      <td>64</td>
      <td>26</td>
      <td>23</td>
      <td>15</td>
      <td>0.406250</td>
      <td>0.234375</td>
      <td>0.359375</td>
      <td>0.061806</td>
      <td>0.053213</td>
      <td>0.060367</td>
      <td>0.182386</td>
      <td>False</td>
    </tr>
    <tr>
      <th>Michigan</th>
      <td>324</td>
      <td>123</td>
      <td>115</td>
      <td>86</td>
      <td>0.379630</td>
      <td>0.265432</td>
      <td>0.354938</td>
      <td>0.027841</td>
      <td>0.025192</td>
      <td>0.027430</td>
      <td>0.081843</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Idaho</th>
      <td>27</td>
      <td>11</td>
      <td>9</td>
      <td>7</td>
      <td>0.407407</td>
      <td>0.259259</td>
      <td>0.333333</td>
      <td>0.094828</td>
      <td>0.084526</td>
      <td>0.090961</td>
      <td>0.277224</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>



In the end, the sentiment analysis improved the average relative error of our team dominance measurement in the following states:


```python
for state in stateInfo.index[stateInfo['Improved'] == True]:
    print (state)
```

    Pennsylvania
    Michigan
    Alabama
    Puerto Rico
    California
    Tennessee
    Illinois
    Nevada
    North Carolina
    South Dakota
    Washington
    Minnesota
    Kentucky
    Florida
    Ohio
    Indiana
    Massachusetts
    New Jersey
    Texas
    Oklahoma
    Georgia
    Arizona
    Virginia
    New York
    Maryland
    Oregon


Now that we've finished our statistical analysis, let's save the data frames so that we can visualize the results in our [next and final blog post](#http://www.raknoche.github.io/2016/08/03/PoGo-Series-Making-a-Choropleth-Map/).


```python
stateInfo.to_csv('stateInfo.csv')
stateInfo_nosenti.to_csv('stateInfo_nosenti.csv')
```
