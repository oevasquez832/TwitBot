

```python
# Dependencies
import tweepy
import twitconfig as cfg
import os
import json
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from datetime import datetime
import time

# Import and Initialize Sentiment Analyzer
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
analyzer = SentimentIntensityAnalyzer()

consumer_key=cfg.api_key
consumer_secret=cfg.api_secret
access_token=cfg.access_token
access_token_secret=cfg.token_secret
          

# Setup Tweepy API Authentication
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_token_secret)
api = tweepy.API(auth, parser=tweepy.parsers.JSONParser())
```


```python
def twit_req(tweet, tweet_dict=dict()):
    tweet_list = []
    tweet_id = tweet["id"]
    tweet_us = tweet["user"]["screen_name"]
    tweet_req = []
    print(tweet_id)
    for tags in tweet["entities"]["user_mentions"]:
        if tags["screen_name"] != "NotAScamBot":
            tweet_req.append(tags["screen_name"])
    
    tweet_dict = {"id":tweet_id,"user":tweet_us,"analysis_requests":tweet_req}
    return tweet_dict
```


```python
def sent_analysis(recent_tweets, sent_result=list()):
    sent_results = []
    for tweet in recent_tweets:
        new_tweet = clean(tweet)
        sent_result = analyzer.polarity_scores(new_tweet["text"])
        sent_result.update({"tweet_id":new_tweet["id"]})
        sent_results.append(sent_result)    
    return sent_results
```


```python
def rm_noise(tweet, category, key, tweet_result=dict()):
    try:
        tweet_result = tweet
        tweet_text = tweet.get("text")
        tweet_stuff = tweet.get("entities").get(category)
        for stuff in tweet_stuff:
            replace_str = stuff[key]
            tweet_text = tweet_text.replace(replace_str," ")
        tweet["text"] = tweet_text
    except TypeError:
        pass
    return tweet_result

```


```python
def clean(tweet,tweet_result=dict()):
    tweet_result = tweet
    tweet_result = rm_noise(tweet_result,"user_mentions","screen_name")
    tweet_result = rm_noise(tweet_result,"urls","url")
    tweet_result = rm_noise(tweet_result,"media","url")
    tweet_result["text"] = tweet_result["text"].replace("@","")
    return tweet_result
```


```python
def plot_sentiments(title,sentiments):
    df = pd.DataFrame(sentiments)
    df = df.reset_index()
    df.plot( 'index', 'compound', linestyle='-', marker='o',alpha=0.50)
    plt.ylabel("Tweet Polarity")
    plt.xlabel("Number of Tweets")
    plt.title(title)
    
    filename = "Sentiment_Analysis_for_"+title+".png"
    plt.savefig(filename)
    
    return filename
```


```python
def look(since_tweet_id):
    handle = "@NotAScamBot"

    res = api.search(handle,since_id = since_tweet_id)

    if(len(res["statuses"]) > 0):
        tweet_list = []

        for tweet in res["statuses"]:
            tweet_list.append(twit_req(tweet))
        
        for item in tweet_list:

            recent_tweets = []

            for analyze_req in item["analysis_requests"]:

                recent_tweets = api.user_timeline(analyze_req,count=200)

                if(len(recent_tweets) > 0):
                    sentiments = sent_analysis(recent_tweets)
                    sentiment_fig = plot_sentiments(analyze_req,sentiments)
                    text_status = f"{datetime.now()} - You only want me for my sentiments @{item['user']}!"
                    api.update_with_media(filename=sentiment_fig,status=text_status,in_reply_to_status_id=item["id"])
                else:
                    text_status = f"{datetime.now()} - No tweets for @{item['user']}! {analyze_request} is a loser!" 
                    api.update_status(text_status)
                time.sleep(300)
                plt.show()
        return res["statuses"][0]["id"]
    else:
        return since_tweet_id
```


```python
since_tweet_id = 971036516947554305
while True:
    since_tweet_id = look(since_tweet_id)
    time.sleep(300)

```
