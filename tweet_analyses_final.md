

```python
#####################################################credentials

import tweepy #https://github.com/tweepy/tweepy
import json
import numpy as np
import matplotlib.pyplot as plt
import datetime
import time

#Twitter API credentials
consumer_key = "GvauzqzfE2o1eqIHxk5X8q38u"
consumer_secret = "cBfUvQ37EbQKPP2FjyhqVxTyCrBqnlEa8bts81C5SWOUDxpXsA"
access_token = "980991539739484160-jivEu1NrRKe7rLsHr3ZBBf2YRtYSWab"
access_token_secret = "GPcMoprjKx5D7lhvA3Xj10diRnhcHBLsowELyfE1EuoHN"



    
    #authorize twitter, initialize tweepy
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_token_secret)
api = tweepy.API(auth)

#########################get the twitter handle to put into the formula
def WhoThere():

    # Twitter Credentials
    auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
    auth.set_access_token(access_token, access_token_secret)
    api = tweepy.API(auth, parser=tweepy.parsers.JSONParser())

    # Search for all tweets
    public_tweets = api.search('@bobbobe37676119', count=1, result_type="recent")

    # Loop through all tweets
    for tweet in public_tweets["statuses"]:
       
        global tweet_id
        global tweet_author
        global target_term
        # Get ID and Author of most recent tweet directed to me
        tweet_id = tweet["id"]
        tweet_author = tweet["user"]["screen_name"]
        tweet_text = tweet["text"]
        target_term = str('@' + tweet['entities']["user_mentions"][1]['screen_name'])
        
        
        # Print the tweet_text
        print(tweet_text)
        print(target_term)
WhoThere


#####################################get all the tweets from the target

def gettweets():

    #target_term = "washingtonpost" 


    #initialize a list to hold all the tweepy Tweets
    global alltweets
    alltweets = []
    
    #make initial request for most recent tweets (200 is the maximum allowed count)
    new_tweets = api.search(target_term, count=100)   #, result_type="recent"
    
    #save most recent tweets
    alltweets.extend(new_tweets)
    
    #save the id of the oldest tweet less one
    oldest = alltweets[-1].id - 1
    
    #keep grabbing tweets until there are no tweets left to grab
    while len(alltweets) < 500:
        #print "getting tweets before %s" % (oldest)
        
        #all subsiquent requests use the max_id param to prevent duplicates
        new_tweets = api.search(target_term, count=100,max_id=oldest)
        
        #save most recent tweets
        alltweets.extend(new_tweets)
        
        #update the id of the oldest tweet less one
        oldest = alltweets[-1].id - 1
        
        #print "...%s tweets downloaded so far" % (len(alltweets))
    print(len(alltweets))

 

############################analyze the tweets
def analyze():

    from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
    analyzer = SentimentIntensityAnalyzer()

    #test = alltweets[0]
    json_str = json.dumps(alltweets[2]._json)     #parse json now that we have the tweets
    #print(json_str)

    #print((alltweets)[1])
    #print(type(alltweets[1]))
    global forx
    global sentiment
    sentiment = []
    forx = []
    tweet_id = []
    jdic = []
    w=1

    jsontweets = []

    for tweet in alltweets:
        #convert to json
        y = json.dumps(tweet._json)
        #y = json.loads(tweet._json)
        jsontweets.append(y)

    for item in jsontweets:
        d = json.loads(item)
        jdic.append(d)
    #print(jdic[0]['text'])
    
    #print(len(jsontweets))
    #print(type(jsontweets[0]))

    #d = json.loads(jsontweets[0])
    #print(d['text'])
    super = []

    print(type(jdic[8]))

  
    compound = analyzer.polarity_scores(jdic[8]["text"])["compound"]
    print(compound)
    i = 0
    texttext=[]
    while i < 500:
    
        # Run Vader Analysis on each tweet
        compound = analyzer.polarity_scores(jdic[i]["text"])["compound"]
        texty = jdic[i]["text"]
        #pos = analyzer.polarity_scores(tweet["text"])["pos"]
        #neu = analyzer.polarity_scores(tweet["text"])["neu"]
        #neg = analyzer.polarity_scores(tweet["text"])["neg"]

        # Add each value to the appropriate array
        sentiment.append(compound)
        texttext.append(texty)
        #sentiment.append(neu)
        #sentiment.append(neg)
        i += 1
        w -= 1
        forx.append(w)
    #print(sentiment)
    #print(texttext)


########################make a dope plot
def plot():
    
    now = datetime.datetime.now() 
    today = now.strftime("%Y-%m-%d")    
    # Create data
    #N = 500
    x = forx
    y = sentiment
    colors = (0,0,0)
    area = np.pi*3
 
    # Plot
    plt.scatter(x, y, s=area, c=colors, alpha=0.5)
    plt.title(f'Sentiment Analysis of {target_term}\'s Tweets {today}')
    plt.xlabel('Tweets Ago')
    plt.ylabel('Tweet Polarity')
    plt.savefig('/Users/justin/Documents/2018/ucsd/tweetanalhw/botplot2.png')
    plt.show()

############################# tweet out the graph and shit
def ThankYou():

    # Twitter Credentials
    auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
    auth.set_access_token(access_token, access_token_secret)
    api = tweepy.API(auth, parser=tweepy.parsers.JSONParser())

    # Search for all tweets
    public_tweets = api.search('@bobbobe37676119', count=1, result_type="recent")

    # Loop through all tweets
    for tweet in public_tweets["statuses"]:
       
        global tweet_id
        global tweet_author
        global target_term
        # Get ID and Author of most recent tweet directed to me
        tweet_id = tweet["id"]
        tweet_author = tweet["user"]["screen_name"]
        tweet_text = tweet["text"]
        target_term = str('@' + tweet['entities']["user_mentions"][1]['screen_name'])
        
        
        # Print the tweet_text
        print(tweet_text)
        print(target_term)

        # Use Try-Except to avoid the duplicate error
        try:
            # Respond to tweet
            api.update_with_media("/Users/justin/Documents/2018/ucsd/tweetanalhw/botplot2.png",
                      f"Hey @{tweet_author}! Here is that graph you wanted for {target_term}!", in_reply_to_status_id=tweet_id)

            # Print success message
            print("Successful response!")

        except Exception:            # Print message if duplicate
            print("Already responded to this one!")

        # Print message to signify complete cycle
        print("We're done for now. I'll check again in 5 minutes.")
        

    
    
    
    





```


```python

while(True):
    WhoThere()    
    gettweets()
    analyze()
    plot()  
    ThankYou()
    time.sleep(300)  
    
    
    
```

    @bobbobe37676119  @just15908306
    @just15908306



    ---------------------------------------------------------------------------

    RateLimitError                            Traceback (most recent call last)

    <ipython-input-14-af9c8fd86767> in <module>()
          2 while(True):
          3     WhoThere()
    ----> 4     gettweets()
          5     analyze()
          6     plot()


    <ipython-input-13-badaa9b42597> in gettweets()
         77 
         78         #all subsiquent requests use the max_id param to prevent duplicates
    ---> 79         new_tweets = api.search(target_term, count=100,max_id=oldest)
         80 
         81         #save most recent tweets


    /anaconda/lib/python3.6/site-packages/tweepy/binder.py in _call(*args, **kwargs)
        248             return method
        249         else:
    --> 250             return method.execute()
        251 
        252     # Set pagination mode


    /anaconda/lib/python3.6/site-packages/tweepy/binder.py in execute(self)
        230 
        231                 if is_rate_limit_error_message(error_msg):
    --> 232                     raise RateLimitError(error_msg, resp)
        233                 else:
        234                     raise TweepError(error_msg, resp, api_code=api_error_code)


    RateLimitError: [{'message': 'Rate limit exceeded', 'code': 88}]



```python

```
