
Creating a sentiment analyzer using Python's Natrual Learning Toolkit.

<!--more-->

TABLE OF CONTENTS

This is the fifth post in an on-going Pokemon Go analysis series.  Last time, we discussed how a Naive Bayes Classifier can be used to predict the class of a sample given a number of features about that sample.  In this post, we'll apply the technique to our Pokemon Go tweets to build a sentiment analyzer that automatically classifies whether each tweet has a positive or negative tone.  Once complete, we'll use the sentiment analyzer to remove negative tweets from our data set before using the positive tweets to map out the dominance of each team in each state.


We'll cover the following topics:

1. [Manually classifying a training set](#labeling)
2. [Training sets, Test sets, and Cross Validation sets](#sets)
3. [Dealing with imbalanced classes](#imbalanced)
4. [Feature Extraction Design](#features)
5. [Feature Extraction Implementation](#featuresCode)
6. [Training and Modifying our sentiment analyzer](#training)
7. [Evaluating our analyzer's performance](#eval)


# <a name="labeling"></a> Manually classying a training set

Recall from our last post that the implementation of a Naive Bayes Classifier requires a training set of samples which have known features and known classes.  Right now, all of the tweets we've collected are unclassified.  Before we create our sentiment analyzer, we'll have to manually label a subset of the tweets as positive or negative to form our training set.  If you want to avoid manually labeling tweets, you could take an [unsuperivsed machine learning approach](https://marcobonzanini.com/2015/05/17/mining-twitter-data-with-python-part-6-sentiment-analysis-basics/) instead, but these typically have poor performance compared to a supervised approach such as Naive Bayes Classifiers.

The process of manually labeling tweets is going to be time consuming and tedious, but we can use Python to make the process a little more bearable.  First, we'll import the Pandas and JSON libraries.


```python
import json
import pandas as pd
```

If you aren't familiar with the Pandas package, I highly recommend watching [Wes McKinney's tutorial](https://github.com/estimate/pandas-exercises) on the library that he created.  Pandas is designed to provide intuitive interactions with tabular data by introducing data frames to Python.  We'll be using it to create some data frames about our Pokemon Go tweets.


```python
def pop_tweets(path):
    #Declare a new data frame with pandas, with some specific column names
    tweets = pd.DataFrame(columns=['screenName','userId','text','location'])

    #Open the text file that contains the tweets we collected
    tweets_file = open(path, "r")
    
    #Read the text file line by line
    for line in tweets_file:
        
        #Load the JSON information
        tweet = json.loads(line)
        
        #If the tweet isn't empty, add it to the data frame
        if ('text' in tweet): 
            tweets.loc[len(tweets)]=[tweet['user']['screen_name'],tweet['user']['id'],tweet['text'],tweet['place']['full_name']]    

    return tweets

PoGo_tweets = pop_tweets('PoGo_USA.json')
```

We can take a peek at the data frame we just created using Pandas' `head` method.


```python
PoGo_tweets.head(n=5)
```

Great! Now that we have all of the tweets that we collected stored in a data frame it's much easier to assess what our data looks like and any problems we might run into.  In particular, look at tweet in row 4 of the data frame:


```python
print(PoGo_tweets['text'][4])
```

This type of tweet will be very hard to classify.  It's clear this Twitter user has an allegiance to Team Valor, but if we want our sentiment analyzer to recognize that it would need to determine that the first half of the tweet has a negative tone, while the second half of the tweet has a positive tone.  This introduces a lot of complexity to the classifier.  It'd be much simplier to have the analyzer classify the entirity of each tweet as positive or negative, so we should just remove complicated, multi-team tweets from our data set.  To do so, we'll add a new column to our data frame that tells us when a tweet talks about multiple teams.


```python
#Remove tweets that discuss two or more teams
for row in range(len(PoGo_tweets)):
    
    #Set three flags that determine if the tweet mentions each team
    has_b=any(word in PoGo_tweets.loc[row,'text'].lower() for word in ['team mystic','#teamblue','#teammystic','#mystic','mystic'])
    has_r=any(word in PoGo_tweets.loc[row,'text'].lower() for word in ['team valor','#teamred','#teamvalor','#valor','valor'])
    has_y=any(word in PoGo_tweets.loc[row,'text'].lower() for word in ['team instinct','#teamyellow','#teaminstinct','#instinct','instinct'])
    
    #If the tweet mentions more than one team, set the new 'multi-team' column to True
    if has_b+has_r+has_y > 1:
        PoGo_tweets.loc[row,'multi-team']=True
    else:
        PoGo_tweets.loc[row,'multi-team']=False    
```

Now that we've added a new column, let's take another look at the data frame.  Note that the new "multi-team" column is appropriately flagging the tweet in the fourth row as being a multi-team tweet.


```python
PoGo_tweets.head(n=5)
```

Removing the multi-team tweets from our data frame is simple.  The data frame can be indexed with logical statements, so we simply reassign our data frame to the subset of data which passes `PoGo_tweets['multi-team'] == False`.


```python
#Applying cut to remove multi-team tweets
PoGo_tweets = PoGo_tweets[PoGo_tweets['multi-team'] == False]
PoGo_tweets.head(n=5)
```

Notice that we're now missing a fourth row in our data frame! Pandas does not automatially reindex the data frame after we apply a logical cut.  We can force it to do so with the `rest_index` method.


```python
PoGo_tweets = PoGo_tweets.reset_index(drop=True)
PoGo_tweets.head(n=5)
```

We'll need to fairly large training set in order for our sentiment analyzer to perform well, but we don't want to spend too much time manually labeling tweets.  After all, if we manually labeled all of the tweets in our collection there would be no need to train a sentiment analyzer.  Manually labeling about 4,000 of the ~20,000 tweets in our collection should be a good middle ground.  We'll begin the process by putting the first 4,000 tweets of our collection into their own data frame.


```python
PoGo_labeled = PoGo_tweets.ix[:4000]
```

I wrote a function to help label all 4,000 tweets quickly.  It loops through each of the rows in our new data frame and displays the tweet on screen.  It then waits for us to read the tweet and manually classify the text as 'pos','neg', or 'nan' (when the tweet is ambiguous) before adding our classification to a new "sentiment" column in the data frame.  Even with this function the process was tedious, and took me about 4 hours to complete on my own.


```python
#Display the tweet for each row, and ask user to label it

#Classifications:
#pos: Can identify the user as being on the team
#neg: Negative tweet about the team
#nan: Can't identify the user as being on the team, but the tweet is not negative

for row in range(4000):
    print (PoGo_labeled.loc[row,'text'])
    PoGo_labeled.loc[row,'sentiment'] = input()
```

After we've finished manually classifying our training set, we should save the data to a csv file so that we don't have to repeat the process.


```python
PoGo_labeled.to_csv('PoGo_Sentiment_Labeled.csv')
```

# Loading data and importing extra stuff... discuss imports at some point



```python

#Import the csv dataframe
#CUT THIS FROM THE FINAL MARKDOWN
PoGo_labeled = pd.read_csv('PoGo_Sentiment_Labeled_extended.csv')
PoGo_labeled.drop('Unnamed: 0', axis=1, inplace=True)
PoGo_labeled.drop('latt', axis=1, inplace=True)
PoGo_labeled.drop('long', axis=1, inplace=True)
PoGo_labeled.head(n=5)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>location</th>
      <th>multi-team</th>
      <th>screenName</th>
      <th>sentiment</th>
      <th>text</th>
      <th>userId</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Caldwell, ID</td>
      <td>False</td>
      <td>desmond_ayala</td>
      <td>pos</td>
      <td>Which pokemon go team did y'all chose? #valor</td>
      <td>2.953472e+09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Brooklyn, NY</td>
      <td>False</td>
      <td>aphrospice</td>
      <td>pos</td>
      <td>#Magikarp practicing his struggle skills in th...</td>
      <td>1.629086e+07</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bixby, OK</td>
      <td>False</td>
      <td>ABellgowan</td>
      <td>pos</td>
      <td>Pokemon Go is taking over my life #TeamInstinct</td>
      <td>1.681036e+09</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Los Angeles, CA</td>
      <td>False</td>
      <td>JangoSnow</td>
      <td>pos</td>
      <td>Go Team Instinct! I like underdogs. :)  https:...</td>
      <td>1.057434e+07</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Niagara Falls, NY</td>
      <td>False</td>
      <td>EmberLighta2</td>
      <td>pos</td>
      <td>#TeamMystic has total control of Niagara Falls!!</td>
      <td>7.513920e+17</td>
    </tr>
  </tbody>
</table>
</div>



# <a name="sets"></a> Training sets, Test sets, and Cross Validation sets


Before we use the training set to build our sentiment analyzer, we need to consider how to properly handle the data.  Suppose we fed all of our training set to the sentiment analyzer, allowing it to extract whatever features were needed to maximize its prediction accuracy.  In this case, we run the risk of over constraining our sentiment analyzer with obscure features that are only relevant to a handful of tweets.  For instance, maybe one of the tweets that was not included in our training set contains the text "I really love #TeamValor.  Psych!"  Without using the fairly uncommon word "psych" as a feature, our analyzer would probably classify this tweet as positive.  The only chance we have of correctly identifying the tweet as negative is if the word "psych" was also use in one of our training set tweets.  There is no chance that our training set contains every word (inlcuding mispellings and acronyms) that will show up in our entire collection of tweets, so if we evaluate the performance of our classifier based on the tweets it was trained on we will likely overestimate its performance.

To solve this problem, we'll set a portion of our original training set aside, not to be touched until we have finished training our sentiment analyzer.  This reserved set of data is known as the **test set**, which we will use to evaluate the performance of our sentiment analyzer once it is complete.

When designing our sentiment analyzer, we'll have a number of choices to make along the way.  How many features should we use?  Should we include the capitalization of words as a feature?  Should we label additional tweets to increase the size of our training set?  These types of choices are known as **hyperparameters** in machine learning, and the answers to each of these questions will effect the performance of our sentiment analyzer.  To improve performance of out sentiment analyzer, we'd like to optimize the hyperparameters we use.  To so do,  we'll need to evaluate the performance of our classifier under different hyperparameter settings &mdash; but which data should we for this evaluation?  We could evaluate the performance using the training set itself, but then we would run into the same risk of over constraining the analyzer that we discussed before.  Instead, we could evaluate the performance of each hyperparameter using the test set that we set aside earlier, but then we would be tuning our classifier to the test set, which defeats the purpose of setting that data aside in the first place.  

To solve this problem, we'll set another portion of the original training set aside.  This new subset of data is called the **cross validation set**.  We can use it to evaluate the performance of our sentiment analyzer as we tune our hyperparameters.  Since the analyzer is not trained on data from the cross validation set, there is no chance that the performance evaluation will be skewed due to over fitting the data.  Likewise, since analyzer is not trained on data from the test set, and since the the hyperparameters were not tuned with data from the test set, our final evaluation of our analyzer's performance will be a true representation of how it will perform on unlabeled tweets from full entire collection.

# <a name="imbalanced"></a> Dealing with imbalanced classes

Before we decide how much data to set aside for our training, test, and cross validation sets, let's take a look at how many tweets we labeled as positive and negative.




```python
#Use list comprehensions to make lists of positive and negative tweets
pos_tweets = [(PoGo_labeled.ix[row,'text'],'positive') for row in range(len(PoGo_labeled)) if \
              PoGo_labeled.ix[row,'sentiment'] == 'pos']

neg_tweets = [(PoGo_labeled.ix[row,'text'],'negative') for row in range(len(PoGo_labeled)) if \
              PoGo_labeled.ix[row,'sentiment'] == 'neg']

print('Number of tweets labeled positive: %d' % len(pos_tweets))
print('Number of tweets labeled negative: %d' % len(neg_tweets))
```

    Number of tweets labeled positive: 3408
    Number of tweets labeled negative: 304


Notice that only about 10% of the tweets are negative.  This class imbalance presents a new challenge for our sentiment analyzer.  Bayes' Theorem tells us that the probability of a tweet belonging to the negative class is

$$\mathsf{ P(Neg | \mathbf{X}) = \frac{ P( \mathbf{X} | \lvert Neg ) P(Neg) }{P(\mathbf{X})}}$$

Since the probability of seeing negative tweets, $$\mathsf{P(Neg)}$$, is so small Bayes Theorem is likely to predict that the probability of class being negative is small regardless of the tweet's features.  In fact, if the classifier predicted that all of the tweets we collected were positive it would be correct 90% of the time!  This isn't a particularly useful classifier though, since it wouldn't help us eliminate negative tweets from our samples at all.

The are a few ways to deal with imbalanced classes.  One option is to **up sample** the the population of negative tweets in our training set.  This means that each time we put a negative tweet into our training set, we actually enter it multiple times.  In this case, we'd have to copy 10 instances of each negative tweet that we put into our training set for the classes to be balanced.  Up sampling allows us to see a larger population of the positive tweets, at the cost of giving each feature that we see in the negative tweets more weight.  This typically works best when we have very little data to work with.

Alternatively, we could **down sample** the population of positive tweets in our training set.  This means that each time we put a negative tweet into our training set, we put one positive tweet in as well.  This will result in having far fewer positive tweets in the training sample than up sampling would, but the weight of negative features will be an exact representation of what we see in our population.  This typically works best when we have a large amount of data to work with.

The choice of up sampling, down sampling, or randomly sampling (producing the true positive to negative ratio) is a hyperparameter of our sentiment analyzer.  The easiest way to know which method will work best is to simply try all three and see which gives the best performance on your cross validation set.  In this case, I found that randomly sampling performed horribly, and down sampling performed slightly better than up sampling.  Below, I've included the code I used to put half of the negative tweets into the training set while down sampling positive tweets at a one-to-one ratio.  Half of the remaining tweets were put into my cross validation set, and the final quarter of tweets were put into the test set.


```python
#half the negative tweets go in training
#Downsampling the positive tweets at 1 pos:1 neg
len_train = int(round(len(neg_tweets)/2)*2)
train_tweets = neg_tweets[:int(len_train/2)] + pos_tweets[:int(len_train/2)]

#half of the remaining half go in cv
cv_neg_cutoff = int( (len_train/2) + round((len(neg_tweets) - len_train/2)/2) )
cv_pos_cutoff = int( (len_train/2) + round((len(pos_tweets) - len_train/2)/2) )
cv_tweets =  neg_tweets[int(len_train/2):cv_neg_cutoff] +  pos_tweets[int(len_train/2):cv_pos_cutoff]  

#rest go into testing
test_tweets = neg_tweets[cv_neg_cutoff:] +  pos_tweets[cv_pos_cutoff:]  
```

#  <a name="features"></a> Feature Extraction Design

We'll define our sentiment analyzer's features with boolean `contains(word)` statements.  For instance, the feature `contains(hate)` will likely be true for a negative tweet, and false for a positive tweet.  We'll need to construct a list of useful words for this purpose.

Our list of words shouldn't include every word we come across in our collection of tweets.  Uncommon words, including acronyms, abbreviations, and mispellings, will cause our sentiment analyzer to overfit the training sample.  We can avoid this issue by requiring that each word appear a certain number of times before we consider it as a feature.  While doing so, we need to consider whether or not to treat words with different capitalizations or punctuations as unqiue.  After testing these hyperparameters with my cross validation set, I found that ignoring capitalization (with the `.lower` string method) and punctuation marks (with the `string.punctuation` set) works best in this case.  Similarly, I found that treating all URLs as identical improved performance on my cross validation set.  Note that in some situations capitalization or punctuation marks may be important context clues, so you'll need to evaluate the hyperparameters on your own cross validation set.


Many words, such as "an", are common to both positive and negative tweets.  Including these words as features can actually decrease the performance of our classifier, since they increase the dimensionality of our model without adding additional information.  There are a few ways to deal with such features:

1. Remove the common stop words.  The NLTK library provides a list of common stop words.  You can import it to Python using:
```python
from nltk.corpus import stopwords
stop = set(stopwords.words('english'))
```
After importing the list of stop words, it's easy to filter them out with an `if ... not in` statement.  

2. Limit our model to [high-information features](http://streamhacker.com/2010/06/16/text-classification-sentiment-analysis-eliminate-low-information-features/).  These features will contain words which show up far more often in one class than the other.  
With this technique we train our model once, and use it to gain information about the most informative features.  We then train our analyzer a second time, using only the most informative features.  The exact number of high information features to use is a hyperparameter which will need to be tuned with the cross validation set.  Using high information features is often a better strategy than simply excluding stop words, since some of the stop words may actually turn out to be important in our context.  This strategy will also help us filter out uncommon words if we neglected to do so beforehand.

I found that my analyzer performed better on my cross validation set when stop words were no excluded as features.  This is likely due to important stop words, such as "my," which help identify which Pokemon Go team the Twitter user is on. I also found that limiting my model to include only high-information features did not significantly change performance, which is likely due to my efforts to optimize the feature extraction process using the other methods discussed in this section.

Now, consider the following two tweets:

1. "I do like #TeamValor."
2. "I do not like #TeamValor."

The content of these tweets are extremely similar.  If we were to train a model that believes the word "like" indicates a positive tweet, we would misclassify the second tweet.  It's clear that the classifier's performance would increase if it could recognize that the word "not" is negating the word "like."  One way to achieve this is to distinguish every word that follows a negation &mdash; in this case we would end up with  `contains(like)` as a feature for the first tweet, and `contains(like_NEG)` as a feature for the second tweet.  NLTK's `mark_negation` function can do this for us:


```python
import nltk
from nltk.sentiment.util import mark_negation

example = "I do not like #TeamValor".split()
nltk.sentiment.util.mark_negation(example)
```




    ['I', 'do', 'not', 'like_NEG', '#TeamValor_NEG']



Another way to address negation markers is to include [bigrams](http://www.nltk.org/howto/collocations.html) in our list of features.  Doing so allows our classifier to recognize the collocation of words such as "(not,like)" in addition to the typical unigram features we would extract.  The inclusion of bigrams also allows our classifier to recognize collocations that do not mark negation, such as "(fuck, team)", which is far more powerful than tracking the common words "fuck" and "team" as unigrams alone. (Pardon my French here, but this turned out to be the strongest example of bigrams features from our tweets.)

Finally, we want to ensure that our sentiment analyzer performs equally well regardless of which Pokemon Go team the tweet is discussing.  If we were to misclassify every negative tweet about Team Valor as positive, but always correctly classify negative tweets about Team Mystic, then our final population of tweets would contain an overabundance of Team Valor tweets.  This would certainly skew the map of team dominance which we hope to make with our data.  To avoid this problem, I've excluded any words that could identify a Pokemon Go team from our list of features.  

#  <a name="featuresCode"></a> Feature Extraction Implementation

I've included the code which applies the feature extraction techniques to our tweets below.  First, we filter the text of each tweet to extract a list of unigrams and bigrams.  I remove any "#" markers from the text, treat all html URLs as the same "word", and exclude punctuation, capitaization, and team-identfying words in the process.


```python
from nltk.metrics import BigramAssocMeasures
from nltk.collocations import BigramCollocationFinder
import string
import itertools


#Set to exclude punctuation marks
exclude = set(string.punctuation)

#List to exclude words that can identify the team from list of features
excluded_words = ['teammystic','mystic','teamblue','blue',\
                  'teaminstinct','instinct','teamyellow','yellow',\
                  'teamvalor','valor','teamred','red']

#Function that provides a list of filtered unigrams and bigrams from each tweet
def filter_tweets(tweets):
    filtered_tweets = []
    
    #Get a list of words, and the sentiment for each tweet
    for (words, sentiment) in tweets: 
        words_filtered=[]
        
        #For each word in the list of words, filter on our feature requirements. 
        for word in words.split(): 
            
            #Remove punctuation
            word = ''.join(ch for ch in word if ch not in exclude)

            #Remove zero letter "words"
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

        #Identify top 200 bigams in the filtered word list using chi_sq measure of importance
        bigram_finder = BigramCollocationFinder.from_words(words_filtered)
        bigrams = bigram_finder.nbest(BigramAssocMeasures.chi_sq, 200)      

        #Add the final bigrams and unigrams for this tweet to the filtered list
        filtered_tweets.append(([ngram for ngram in itertools.chain(words_filtered, bigrams)],sentiment))

    #Returnt he filtered list for all tweets
    return filtered_tweets
```


```python
#Filter each set of data
train_tweets = filter_tweets(train_tweets)
cv_tweets = filter_tweets(cv_tweets)
test_tweets = filter_tweets(test_tweets)    
```

Once we have a list of filtered unigrams and bigrams from each tweet, I combine all of the unigrams and bigrams from our training set into one list.  If a unigram or bigram appears more than 3 times in this list, I include it as a feature.


```python
#Function that builds a list of features from the list of unigrams and bigrams
#Requires each unigram or bigram to show up some minimum number of times to be considered a feature
def get_word_features(tweets,min_freq):

    #Create a list of ALL unigrams and bigrams
    wordlist = []
    for (words, sentiment) in tweets:
        wordlist.extend(words)
    
    #Count the frequency of each unigram and bigram
    wordlist = nltk.FreqDist(wordlist)
    
    #Sort the list of unigrams and bigrams based on frequency
    sorted_word_list = sorted(wordlist.items(), key=lambda x: x[1], reverse=True)
    
    #Only include the unigrams and bigrams as features if they appear at least min_freq times
    word_features = [sorted_word_list[word][0] for word in range(len(sorted_word_list)) if sorted_word_list[word][1] >= min_freq]
    
    #Return the list of features
    return word_features

word_features = get_word_features(train_tweets,3)
```

Now that we have a final list of word features, we'll want evaluate the boolean `contains(word_feature)` features for each tweet.  To do so, we first define an `extract_features` function that takes a filtered tweet as an input, then determines if each word feature exists in the filtered tweet's list of unigrams and bigrams.


```python
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

The NLTK classify package provides a convenient `.apply_features` method to apply our `extract_features` function to each tweet in our data sets:


```python
#Extract features from each tweets
training_set = nltk.classify.apply_features(extract_features, train_tweets)
cv_set = nltk.classify.apply_features(extract_features, cv_tweets)
test_set = nltk.classify.apply_features(extract_features, test_tweets)
```

The final result is a NLTK LazyMap object that contains a tuple for each Tweet.  The first entry in the tuple is a list of boolean `contains(word)` features for each of the feature words we selected, while the second entry in the tuple is the manually labeled sentiment of the tweet.


```python
tweet_number=1
training_set[tweet_number][0]
```




    {"contains(('a', 'gym'))": False,
     "contains(('a', 'joke'))": False,
     "contains(('a', 'pokemon'))": False,
     "contains(('a', 'team'))": False,
     "contains(('all', 'the'))": False,
     "contains(('and', 'i'))": False,
     "contains(('are', 'you'))": False,
     "contains(('at', 'the'))": False,
     "contains(('but', 'i'))": False,
     "contains(('chose', 'team'))": False,
     "contains(('for', 'team'))": False,
     "contains(('friends', 'anymore'))": False,
     "contains(('fuck', 'team'))": False,
     "contains(('go', 'team'))": False,
     "contains(('going', 'to'))": False,
     "contains(('gym', 'at'))": False,
     "contains(('gym', 'http'))": False,
     "contains(('i', 'am'))": False,
     "contains(('i', 'dont'))": False,
     "contains(('i', 'know'))": False,
     "contains(('i', 'was'))": False,
     "contains(('if', 'you'))": False,
     "contains(('if', 'youre'))": False,
     "contains(('im', 'just'))": False,
     "contains(('in', 'the'))": False,
     "contains(('is', 'a'))": False,
     "contains(('is', 'on'))": False,
     "contains(('is', 'team'))": False,
     "contains(('is', 'the'))": False,
     "contains(('its', 'a'))": False,
     "contains(('join', 'team'))": False,
     "contains(('like', 'team'))": False,
     "contains(('more', 'like'))": False,
     "contains(('my', 'friends'))": False,
     "contains(('of', 'the'))": False,
     "contains(('on', 'my'))": False,
     "contains(('on', 'team'))": False,
     "contains(('on', 'the'))": False,
     "contains(('over', 'the'))": False,
     "contains(('pokemon', 'go'))": False,
     "contains(('pokemongo', 'http'))": False,
     "contains(('taking', 'over'))": False,
     "contains(('team', 'and'))": False,
     "contains(('team', 'are'))": False,
     "contains(('team', 'but'))": False,
     "contains(('team', 'has'))": False,
     "contains(('team', 'http'))": False,
     "contains(('team', 'i'))": False,
     "contains(('team', 'im'))": False,
     "contains(('team', 'is'))": False,
     "contains(('team', 'more'))": False,
     "contains(('team', 'pokemongo'))": False,
     "contains(('team', 'we'))": False,
     "contains(('that', 'team'))": False,
     "contains(('the', 'fuck'))": False,
     "contains(('the', 'gym'))": False,
     "contains(('the', 'only'))": False,
     "contains(('the', 'team'))": False,
     "contains(('this', 'is'))": False,
     "contains(('to', 'be'))": False,
     "contains(('to', 'take'))": False,
     "contains(('what', 'team'))": False,
     "contains(('when', 'you'))": False,
     "contains(('with', 'the'))": False,
     "contains(('would', 'be'))": False,
     "contains(('youre', 'team'))": False,
     "contains(('😂😂', 'http'))": False,
     'contains(1)': False,
     'contains(2)': False,
     'contains(3)': False,
     'contains(a)': False,
     'contains(about)': False,
     'contains(all)': False,
     'contains(also)': False,
     'contains(am)': False,
     'contains(amp)': False,
     'contains(and)': False,
     'contains(any)': False,
     'contains(anymore)': False,
     'contains(are)': False,
     'contains(arent)': False,
     'contains(as)': False,
     'contains(at)': False,
     'contains(back)': False,
     'contains(be)': True,
     'contains(beat)': False,
     'contains(because)': False,
     'contains(been)': False,
     'contains(best)': False,
     'contains(but)': False,
     'contains(can)': False,
     'contains(cant)': False,
     'contains(catch)': False,
     'contains(choose)': False,
     'contains(chose)': False,
     'contains(come)': False,
     'contains(control)': False,
     'contains(cool)': False,
     'contains(day)': False,
     'contains(did)': False,
     'contains(do)': False,
     'contains(does)': False,
     'contains(doesnt)': False,
     'contains(dont)': False,
     'contains(down)': False,
     'contains(even)': False,
     'contains(every)': False,
     'contains(everyone)': False,
     'contains(find)': False,
     'contains(for)': False,
     'contains(friend)': False,
     'contains(friends)': False,
     'contains(from)': False,
     'contains(fuck)': False,
     'contains(fucking)': False,
     'contains(game)': False,
     'contains(get)': False,
     'contains(go)': False,
     'contains(going)': False,
     'contains(good)': False,
     'contains(got)': False,
     'contains(great)': False,
     'contains(guy)': False,
     'contains(gym)': False,
     'contains(gyms)': False,
     'contains(had)': False,
     'contains(has)': False,
     'contains(hate)': False,
     'contains(have)': False,
     'contains(he)': False,
     'contains(help)': False,
     'contains(her)': False,
     'contains(here)': False,
     'contains(hes)': False,
     'contains(hey)': False,
     'contains(how)': False,
     'contains(http)': False,
     'contains(i)': False,
     'contains(if)': False,
     'contains(im)': False,
     'contains(in)': False,
     'contains(is)': False,
     'contains(it)': False,
     'contains(its)': False,
     'contains(ive)': False,
     'contains(join)': False,
     'contains(joined)': False,
     'contains(joke)': False,
     'contains(just)': False,
     'contains(know)': False,
     'contains(last)': False,
     'contains(lets)': False,
     'contains(level)': False,
     'contains(life)': False,
     'contains(like)': False,
     'contains(line)': False,
     'contains(lmao)': False,
     'contains(lockdown)': False,
     'contains(lol)': False,
     'contains(love)': False,
     'contains(me)': False,
     'contains(mom)': False,
     'contains(more)': False,
     'contains(morning)': False,
     'contains(my)': False,
     'contains(never)': False,
     'contains(no)': False,
     'contains(not)': False,
     'contains(now)': False,
     'contains(of)': False,
     'contains(officially)': False,
     'contains(on)': False,
     'contains(one)': False,
     'contains(only)': False,
     'contains(or)': False,
     'contains(our)': False,
     'contains(out)': False,
     'contains(over)': False,
     'contains(park)': False,
     'contains(part)': False,
     'contains(people)': False,
     'contains(picked)': False,
     'contains(pokemon)': False,
     'contains(pokemongo)': False,
     'contains(pokestop)': False,
     'contains(pokémon)': False,
     'contains(reason)': False,
     'contains(right)': False,
     'contains(saying)': False,
     'contains(says)': False,
     'contains(see)': False,
     'contains(shit)': False,
     'contains(so)': True,
     'contains(some)': False,
     'contains(someone)': False,
     'contains(sorry)': False,
     'contains(still)': False,
     'contains(sucks)': False,
     'contains(take)': False,
     'contains(taking)': False,
     'contains(team)': True,
     'contains(tell)': False,
     'contains(than)': False,
     'contains(that)': False,
     'contains(thats)': False,
     'contains(the)': False,
     'contains(then)': False,
     'contains(there)': False,
     'contains(they)': False,
     'contains(think)': False,
     'contains(this)': False,
     'contains(though)': False,
     'contains(time)': False,
     'contains(to)': False,
     'contains(today)': False,
     'contains(too)': False,
     'contains(took)': False,
     'contains(town)': False,
     'contains(trash)': False,
     'contains(trust)': False,
     'contains(type)': False,
     'contains(u)': False,
     'contains(up)': False,
     'contains(ur)': False,
     'contains(usually)': False,
     'contains(wanna)': False,
     'contains(was)': False,
     'contains(we)': False,
     'contains(well)': False,
     'contains(what)': False,
     'contains(when)': False,
     'contains(where)': False,
     'contains(who)': False,
     'contains(why)': False,
     'contains(will)': False,
     'contains(with)': False,
     'contains(work)': False,
     'contains(would)': False,
     'contains(yall)': False,
     'contains(yo)': False,
     'contains(you)': False,
     'contains(your)': False,
     'contains(youre)': False,
     'contains(😂)': True,
     'contains(😂😂)': False,
     'contains(😉)': False,
     'contains(🙄)': False}



# <a name="training"></a> Training and Modifying our sentiment analyzer

Once we've evaluated the `contains(feature_word)` feature statements for each of our tweets,  we can train a Naive Bayes Classifier with one line of code:


```python
#Train the classifier
classifier = nltk.NaiveBayesClassifier.train(training_set)
```

Recall from our discussion of [imbalanced classes](#imbalanced) that only 10% of our tweets are labeled as negative.  This means that our classifier could predict a every tweet as positive and maintain 90% accuracy.  Clearly, such a classifier would be useless for our purposes, since it would fail to identify any of the negative tweet that we wish to remove from our population.  To accurately evaluate the performance of our classifier, we must use more informative metrics known as **precision** and **recall**.

The **precision** of our classifier is the fraction of samples it correctly identified as positive out of all of the samples that it identified as positive.  Mathematically, this is given by:

$$ \mathsf{Precision = \frac{number \; of \; true \; positives}{number \; of \; true \; positives + number \; of \; false \; positives} } $$ 
  
The **recall** of our classifier is the fraction of samples that were correctly identified as positive out of the total number of positive samples.  If our classifier always predicted the positive class, the recall for the negative class will be zero even though the accuracy would be around 90%.  Mathematically, the recall of our classifier is given by:

$$ \mathsf{Recall = \frac{number \; of \; true \; positives}{number \; of \; true \; positives + number \; of \; false \; negatives)} }$$

If our classifier has high precision for our positive class, we can be confident that the majority of tweets that were  classified as positive really were positive.  If our classifier has high recall for positive classes, we can be confident that the majority of positive tweets were correctly classified as positive.   Often, a change in hyperparameters will result in an increase in precision and decrease in recall, or vice versa.  To balance this trade off, we can average the precision and recall metrics using their [harmonic mean](https://en.wikipedia.org/wiki/Harmonic_mean#Harmonic_mean_of_two_numbers), producing a quantity known as the **F1 score**: 

$$ \mathsf{F1 = 2* \frac{P*R}{P+R} }$$

We'll be using our sentiment analyzer to remove negative tweets that are "contaminating" our collection of tweets.  This means that we want high recall for our negative class so that we correctly identify and reject negative tweets often.  This process lowers the systematic error from negative tweets leaking into our collection.  However, if we incorrectly classify positive tweets as negative too often, we will drastically reduce our sample size and increase the statistical errors of our measurement.  Therefore, we also want our classifier to have high positive recall.  I used the fraction of negative in our entire collection as a custom metric to track this balance of positive recall and negative recall.  I'll refer to this metric as the **negative tweet contamination** (NTC), of our tweet collection.

Using the following notation:

$$\mathsf{number \; of \; initial \; negative \; tweets \equiv  N^-}$$
$$\mathsf{number \; of \; initial \; positive \; tweets \equiv  N^+}$$
$$\mathsf{negative \; recall \equiv R^-}$$
$$\mathsf{positive \; recall \equiv R^+}$$

The intital negative tweet contamination is given by 

$$ \mathsf{ NTC_i = \frac{N^-}{N^- + N^+} }$$

After removing tweets classified as negative from our collection, the final negative tweet contamination is given by:

$$ \mathsf{ NTC_f = \frac{N^- * (1-R^-)}{N^- *(1-R^-1) + N^+* R^+} }$$

Therefore, we can see how much the sentiment analyzer has improved our negative tweet contamination using:

$$\mathsf{ Percent\; Improvement = 100 * \frac{NTC_f - NTC_i}{NTC_i} = 100* \left( \frac{N^+ + N^-}{N^-} \right) \left( \frac{N^-}{N^- + N^+} - \frac{N^- (1-R^-)}{N^- (1-R^-) + N^+ * R^+ }\right) }$$

Before we touch our test set, we'll want to evaluate the metrics we discussed using our cross validation set. We can do so by making a set of positive and negative tweets, as well as positive and negative predictions for those tweets.  Feeding these two sets into the `fmeas`, `prec`, and `rec` methods from NLTK's metrics pacakage will return the values of the standard metrics, as shown below:


```python
from nltk.metrics import precision as prec
from nltk.metrics import recall as rec
from nltk.metrics import f_measure as fmeas
import collections

#Function to evaluate our classifier on the given subset of data
def eval_classifier(data_set):

    #Use .accuracy method to calculate accuracy
    cross_valid_accuracy = nltk.classify.accuracy(classifier, data_set)

    #Create two sets which we'll use to count positive and negative tweets
    ref_set = collections.defaultdict(set)
    obs_set = collections.defaultdict(set)


    #Loop over each tweet in our cross validation set
    for i, (feats, label) in enumerate(data_set):

        #Classify the tweet by feeding the classifier the tweet's features
        observed = classifier.classify(feats)

        #Add the current tweet to the "reference" set under the actual class
        ref_set[label].add(i)

        #Add the current tweet to the "observation" set under the predicited class
        obs_set[observed].add(i)


    #Calculate F score, precision, an recall for positive and negative labels
    #Also calculate accuracy and NTC improvement
    print ('Accuracy:', cross_valid_accuracy)
    print ('F-measure [negative]:', fmeas(ref_set['negative'], obs_set['negative']))
    print ('F-measure [positive]:', fmeas(ref_set['positive'], obs_set['positive']))
    print ('Precision [negative]:', prec(ref_set['negative'], obs_set['negative']))
    print ('Precision [positive]:', prec(ref_set['positive'], obs_set['positive']))
    rec_neg=rec(ref_set['negative'], obs_set['negative'])
    rec_pos=rec(ref_set['positive'], obs_set['positive'])
    print ('Recall [negative]:', rec_neg)
    print ('Recall [positive]:', rec_pos)
    total_neg=len(neg_tweets)
    total_pos=len(pos_tweets)
    ntc_improvement = 100*((total_pos + total_neg)/total_neg)*( (total_neg/(total_neg+total_pos)) - (total_neg*(1-rec_neg))/(total_neg*(1-rec_neg) + total_pos*rec_pos))
    print ('Negative contamination improved by ', ntc_improvement, 'percent')
    
```


```python
eval_classifier(cv_set) 
```

    Accuracy: 0.7775821596244131
    F-measure [negative]: 0.2495049504950495
    F-measure [positive]: 0.8694454013089906
    Precision [negative]: 0.14685314685314685
    Precision [positive]: 0.9898039215686274
    Recall [negative]: 0.8289473684210527
    Recall [positive]: 0.7751842751842751
    Negative contamination improved by  76.42955058361015 percent


At this point, if we are unhappy with the outcome of our performance metrics we would need to change the hyperparameters of our classifier and revaluate its performance on our cross validation set.  As I mentioned in the [feature extraction section](#features), if your algorithm is not performing well you may want to consider limiting your classifier to high-information features.  In my case, my cross validation set said this did not change the performance of the classifier, but in case you need it NLTK's `show_most_informative_features` method will tell which which features are the most informative:


```python
#Show the 5 most important features of our classifier
print (classifier.show_most_informative_features(5))
```

    Most Informative Features
    contains(('team', 'is')) = True           negati : positi =     10.3 : 1.0
                contains(at) = True           positi : negati =      9.0 : 1.0
              contains(when) = True           negati : positi =      8.3 : 1.0
              contains(fuck) = True           negati : positi =      7.7 : 1.0
    contains(('on', 'team')) = True           negati : positi =      7.0 : 1.0
    None


I spent an evening tweaking the hyperparameters of my classifier, and the time invested really paid off. For perspective, the initial version of my classifier did terrible job at recognizing negative tweets, with the cross validation set returning the following metrics:

```
Accuracy: 0.8807471264367817
F-measure [negative]: 0.20192307692307693
F-measure [positive]: 0.9355590062111802
Precision [negative]: 0.22340425531914893
Precision [positive]: 0.9283513097072419
Recall [negative]: 0.18421052631578946
Recall [positive]: 0.9428794992175273
Negative contamination improved by  11.48839202570476 percent
```

This final version of my classifier is 78% accurate.   From the high negative recall, we can see that only 17% of negative tweets will be misclassified as positive. This means we'll be able to elliminate a large portion of the negative tweets from our sample with this classifier.  The positive recall remains high as well, at 78%.  This means that we'll retain a large portion of our sample size once we apply our classifier.  The balance of these two statements is reflected by the fact that our negative contamination metric shows an improvement of 76%.  

Notice that our classifier has low negative precision.  While this isn't ideal, it isn't surprising either.  The low negative precision means we are identifying a lot of tweets as negative when they are actually positive.  Given the large class imbalance in our cross valdiation set, it is not surpising that many of the negatively classified tweets are actually positive tweets.  In statistics, this type of "false positive" (where here, a "positive" result is the tweet being classified as negative) is known as a Type I error.  Although I'd like higher negative precision, a Type I error is okay for our purposes.  As long as the chances of misclassifying a positive tweet are equal for all Pokemon Go teams, a Type I error simply means we are being overly conservative with our classification model. 

As a side note, suppose we wanted to look at which Pokemon Go team was the most hated team.  We could do so by looking at the number of negative tweets about each team.   Unfortunately, the low negative precision means the majority of the "negative" tweets we identify will not actually be negative. Therefore, this sentiment analyzer would be unsuited for such an analysis.

# <a name="eval"></a> Evaluating our analyzer's performance 

After optimizing our hyperparameters with our cross validation set, we need to save our sentiment analyzer and our list of features for future use.  We can easily do so using Python's `pickle` library:


```python
import pickle

#Save the classifier for later use
f = open('PoGo_tweet_classifier.pickle', 'wb')
pickle.dump(classifier, f)
f.close()

#Save document_words as well
with open('PoGo_classifier_feats.pickle', 'wb') as f:
    pickle.dump(word_features, f)
```

Now we can finally crack open our test set and see how our classifier performs on a truely random sample of tweets.  We can use the same `eval_classifier` function that we wrote in the previous section for this purpose:


```python
eval_classifier(test_set)
```

    Accuracy: 0.7834507042253521
    F-measure [negative]: 0.24539877300613497
    F-measure [positive]: 0.8735868448098665
    Precision [negative]: 0.14527845036319612
    Precision [positive]: 0.9876065065840434
    Recall [negative]: 0.7894736842105263
    Recall [positive]: 0.7831695331695332
    Negative contamination improved by  71.406449288021 percent


After a lot of hard work, we have a sentiment analyzer that will remove 78.9% of our negative tweets while retaining 78.3% of our positive tweets.  This will lower the systematic error when counting positive tweets from 8.19% to 2.34%, an improvement of 71.4%!


```python
total_neg=len(neg_tweets)
total_pos=len(pos_tweets)


print('Original systematic error from uncut negative tweets was: ', \
      round(10000*(total_neg/(total_neg + total_pos)))/100, 'percent')
```

    Original systematic error from uncut negative tweets was:  8.19 percent



```python
print('Improved systematic error from uncut negative tweets is: ', \
      round((1-0.714)*10000*(total_neg/(total_neg + total_pos)))/100, 'percent')
```

    Improved systematic error from uncut negative tweets is:  2.34 percent

