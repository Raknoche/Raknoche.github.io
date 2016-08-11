---
layout: post
title: PoGo Series&#58; The Twitter API
comments: true
---

An introduction to Twitter's API.

<!--more-->

TABLE OF CONTENTS

Welcome to the second post in the Pokemon Go analysis series.  Recall that our goal is to construct a map of the dominance of each Pokemon Go team within each state.  The first step of this process is to collect tweets about each team from Twitter.  To do so, we'll need to interact with Twitter's Application Programming Interface ([API](https://www.quora.com/What-is-an-API-4)).

In this post, we'll discuss the following topics:

1. [Getting access to the Twitter API](#access)
2. [Understanding access keys](#keys)
3. [Application-only authentication](#app-only)
4. [Twitter's REST API](#restAPI)
5. [Rate Limits](#rate)
6. [Twitter's Streaming API](#streamAPI)



# <a name="access"></a>Getting access to the twitter API:


Getting access to the Twitter API is simple:

1. Go to [http://apps.twitter.com](http://apps.twitter.com)
2. Log in with your twitter account
3. Click "Create New App"
4. Fill out the information sheet and create the application.  If you don't have a website, just fill that section in with a made up URL.
5. After creating the app, click on "manage keys and access tokens" under "Application Settings."
6. Take note of the consumer API key and the consumer secret.  We'll need these to connect to the API later. Do not share these numbers with anybody else.
6. Click on "create my access token" and take note of the access token and access token secret that are generated.  We'll also need these to connect to the Twitter API.  Do not share these numbers with anybody else.
7. Note that you can change app permissions from this page to allow access to features like direct messaging. The standard read and write permissions are fine for our purposes.

# <a name="keys"></a>Understanding access keys

To understand what the consumer key, consumer secret, access token, and access token secret are we need to discuss some of the security protocols of the Twitter API.  Suppose we build an application that requires information from our user’s Twitter accounts.  For instance, maybe we made an application that takes pictures and we want our users to be able to upload those pictures to Twitter with the push of a button.  We could ask the users to provide their username and password for us.  However, doing so is not safe as it would provide the application with unlimited access to the user’s Twitter account.  Instead, we can generate a special key that provides <i>limited</i> access to the user’s account.  To do so, our application (know as "the client") first asks the user (known as the "resource owner") for permission to use such a key.  When we get the permission we forward it to an authentication server, which returns an access token.  The application can then use the access token to get limited access to Twitter's resource server on the user’s behalf.  This process, known as the [OAuth2.0 authentication protocol](https://tools.ietf.org/html/rfc6749#section-1.3), is depicted below:

![OAuth2.0 Protocol](https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/Oauth2p0_Protocol.png)

The consumer key and consumer secret that are created with our application are login credentials that are used to authenticate our application when we access Twitter's API.  Once our application is authenticated, the access token and access token secret allow it to access Twitter's resource server on behalf of our personal account. 


# <a name="app-only"></a> Application-only authentication
It's possible to make requests to the resource server on behalf of the application itself, rather than on behalf of the application’s users.  Since there is no context of a user, you won’t be able to interact with user specific API features such as posting tweets or updating a user profile.  However, you will still be able to interact with non-user specific API features such as searching for tweets, or retrieving user information.  The trade off for this lower functionality is a higher rate limit when querying the API.  We’ll discuss rate limits in more detail later in this post.  For more information on application-only authentication, visit [Twitter's Dev page](https://dev.twitter.com/oauth/application-only).


# <a name="restAPI"></a> Twitter's REST API

Now that our application has the proper credentials to access Twitter's API, we're ready to discuss details of the API itself.  Twitter actually has a number of APIs which we can interface with.  These APIs fall under two categories &mdash; the REST APIs and the Streaming API.  


The [REST APIs](https://dev.twitter.com/rest/public) provide read and write access to existing Twitter data.  There are a lot of useful APIs in this collection, but we only need to discuss the [Search API](https://dev.twitter.com/rest/public/search) for our purposes.  The Search API behaves similarly to the search bar on the Twitter website.  It is used to search a random sample of the past week of tweets.  Note that the Search API does not capture every tweet in the past week &mdash; it aims for “relevance, not completeness.”  Unfortunately there is no way to access the full database of tweets, or tweets older than a week, without purchasing data from third party companies such as [GNIP](https://www.gnip.com/). 

# <a name="rate"></a> Rate limits

The REST API enforces a [rate limit](https://dev.twitter.com/rest/public/rate-limiting) on the number of queries that can be sent within a 15 minute window.  With user authentication, requests to the API are limited on a per-user basis.  That is, if the query allows for 15 requests in 15 minutes, it will allow 15 requests in 15 minutes for each user (or rather, each access token) using your application.  When using application-only authentication, the requests to the API are limited on a per-application basis.  That is, if the query allows for 15 requests in 15 minutes, it really means a total of 15 requests in 15 minutes.  As mentioned previously, one of the benefits of application-only authentication is that the [rate-limit restrictions](https://dev.twitter.com/rest/public/rate-limits
) are often higher than they are with user authentication.  For the search API (GET search/tweets) we can send 450 queries every 15 minutes with application-only authentication, whereas we can only send 180 queries per user every 15 minutes with user authentication. Since we only have one access token, and since we don't need to perform any user-specific actions, we’ll want to use application-only authentication when searching for tweets.


# <a name="streamAPI"></a> Twitter's Streaming API


If we want to collect tweets in real-time, we would need to use Twitter's [Streaming API](https://dev.twitter.com/streaming/overview).  While the REST API pulls data from Twitter's database, the Streaming API pushes messages to a session.  This allows the Streaming API to download data faster than the REST API can.  Twitter users generate about 6,000 tweets per second, creating what is known as the Twitter [firehose](https://dev.twitter.com/streaming/firehose).  Few applications are granted full access to firehose, with the default Streaming API collecting a random sample of just 1% of those tweets.  You can request elevated access to a larger portion of the firehose by contacting [Twitter’s API support](https://dev.twitter.com/overview/general/support), but most reports suggest you will need to purchase this access from a third-party provider such as [GNIP](https://www.gnip.com/).  Note that Twitter automically performs any parsing, filtering, and aggregation that is needed before returning streaming results.  


# Closing remarks

We've built a solid understanding of the Twitter API, how to obtain access to it, and the options that the Search API and Streaming API provide us with when collecting tweets.  In our next post, we'll discuss how to interface with the Twitter API through Python's Tweepy library.