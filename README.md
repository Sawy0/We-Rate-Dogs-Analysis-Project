# We_Rate_Dogs_Analysis_Project
### I gathered 2.3K tweets from Twitter posted by a page called WeRateDogs using Tweepy. This page rates people's dogs in a funny way and it really publish a hilarious content.
### That was originally a project for a Udacity NAND in the Data Analysis track and they helped with providing a TSV file which was an outcome of a neural network to help identifying the dogs' breeds by photo.
### I cleaned the data I gathered and the data they provided and ran an analysis over them and came up with some pretty cool insights.
### Check the code below, and check the reports in the repo. :D

# Data Wrangling For #WeRateDogs

# Gathering Data


```python
import pandas as pd
import json
import requests
import os
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline
%matplotlib notebook
```


###Listing the download links to all the requried documents for the project.
```python
links = ['https://video.udacity-data.com/topher/2018/November/5bf60fbf_twitter-archive-enhanced/twitter-archive-enhanced.csv',
        'https://video.udacity-data.com/topher/2018/November/5bf60fda_tweet-json/tweet-json',
        'https://video.udacity-data.com/topher/2018/November/5bf60fe7_image-predictions/image-predictions.tsv',
        'https://video.udacity-data.com/topher/2018/November/5bfc35b7_twitter-api/twitter-api.rtf',
        'https://video.udacity-data.com/topher/2018/November/5be5fb4c_twitter-api/twitter-api.py']
```


###Getting the requests from the above links and writing them to the workspace directory.
```python
for link in links:
    response = requests.get(link)
    with open(os.path.join('/home/workspace',link.split("/")[-1]), mode = 'wb') as file:
        file.write(response.content)
```

###Loading DataFrames
```python
archive_df = pd.read_csv("twitter-archive-enhanced.csv")
image_predictions_df = pd.read_csv("image-predictions.tsv", sep='\t')
```

###Initializing Tweepy
```python
# %load 'twitter-api.py'
import tweepy
from tweepy import OAuthHandler
import json
from timeit import default_timer as timer

# Query Twitter API for each tweet in the Twitter archive and save JSON in a text file
# These are hidden to comply with Twitter's API terms and conditions
consumer_key = 'Hidden'
consumer_secret = 'Hidden'
access_token = 'Hidden'
access_secret = 'Hidden'

auth = OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_secret)

api = tweepy.API(auth, wait_on_rate_limit=True)

# NOTE TO STUDENT WITH MOBILE VERIFICATION ISSUES:
# df_1 is a DataFrame with the twitter_archive_enhanced.csv file. You may have to
# change line 17 to match the name of your DataFrame with twitter_archive_enhanced.csv
# NOTE TO REVIEWER: this student had mobile verification issues so the following
# Twitter API code was sent to this student from a Udacity instructor
# Tweet IDs for which to gather additional data via Twitter's API
tweet_ids = archive_df.tweet_id.values
len(tweet_ids)

# Query Twitter's API for JSON data for each tweet ID in the Twitter archive
count = 0
fails_dict = {}
start = timer()
# Save each tweet's returned JSON as a new line in a .txt file
with open('tweet_json.txt', 'w') as outfile:
    # This loop will likely take 20-30 minutes to run because of Twitter's rate limit
    for tweet_id in tweet_ids:
        count += 1
        print(str(count) + ": " + str(tweet_id))
        try:
            tweet = api.get_status(tweet_id, tweet_mode='extended')
            print("Success")
            json.dump(tweet._json, outfile)
            outfile.write('\n')
        except tweepy.TweepError as e:
            print("Fail")
            fails_dict[tweet_id] = e
            pass
end = timer()
print(end - start)
print(fails_dict)

```

###Making a DataFrame from the tweets gathered in the json file.
```python
df_list = []

with open('tweet_json.txt', 'r') as file:
    for line in file:
        tweet = json.loads(line)
        tweet_id = tweet['id']
        retweet_count = tweet['retweet_count']
        fav_count = tweet['favorite_count']
        df_list.append({'tweet_id':tweet_id,
                       'retweet_count': retweet_count,
                       'favorite_count': fav_count})
        
api_df_now = pd.DataFrame(df_list)
```

# Assessing the data programmatically


```python
archive_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 16:23:56 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Phineas. He's a mystical boy. Only eve...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892420643...</td>
      <td>13</td>
      <td>10</td>
      <td>Phineas</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 00:17:27 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Tilly. She's just checking pup on you....</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892177421...</td>
      <td>13</td>
      <td>10</td>
      <td>Tilly</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-31 00:18:03 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Archie. He is a rare Norwegian Pouncin...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891815181...</td>
      <td>12</td>
      <td>10</td>
      <td>Archie</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-30 15:58:51 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Darla. She commenced a snooze mid meal...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891689557...</td>
      <td>13</td>
      <td>10</td>
      <td>Darla</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-29 16:00:24 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Franklin. He would like you to stop ca...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891327558...</td>
      <td>12</td>
      <td>10</td>
      <td>Franklin</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
archive_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2356 entries, 0 to 2355
    Data columns (total 17 columns):
     #   Column                      Non-Null Count  Dtype  
    ---  ------                      --------------  -----  
     0   tweet_id                    2356 non-null   int64  
     1   in_reply_to_status_id       78 non-null     float64
     2   in_reply_to_user_id         78 non-null     float64
     3   timestamp                   2356 non-null   object 
     4   source                      2356 non-null   object 
     5   text                        2356 non-null   object 
     6   retweeted_status_id         181 non-null    float64
     7   retweeted_status_user_id    181 non-null    float64
     8   retweeted_status_timestamp  181 non-null    object 
     9   expanded_urls               2297 non-null   object 
     10  rating_numerator            2356 non-null   int64  
     11  rating_denominator          2356 non-null   int64  
     12  name                        2356 non-null   object 
     13  doggo                       2356 non-null   object 
     14  floofer                     2356 non-null   object 
     15  pupper                      2356 non-null   object 
     16  puppo                       2356 non-null   object 
    dtypes: float64(4), int64(3), object(10)
    memory usage: 313.0+ KB
    


```python
archive_df.shape
```




    (2356, 17)




```python
list(archive_df.columns)
```




    ['tweet_id',
     'in_reply_to_status_id',
     'in_reply_to_user_id',
     'timestamp',
     'source',
     'text',
     'retweeted_status_id',
     'retweeted_status_user_id',
     'retweeted_status_timestamp',
     'expanded_urls',
     'rating_numerator',
     'rating_denominator',
     'name',
     'doggo',
     'floofer',
     'pupper',
     'puppo']




```python
api_df_now.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2331 entries, 0 to 2330
    Data columns (total 3 columns):
     #   Column          Non-Null Count  Dtype
    ---  ------          --------------  -----
     0   tweet_id        2331 non-null   int64
     1   retweet_count   2331 non-null   int64
     2   favorite_count  2331 non-null   int64
    dtypes: int64(3)
    memory usage: 54.8 KB
    


```python
api_df_now.shape
```




    (2331, 3)




```python
image_predictions_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>jpg_url</th>
      <th>img_num</th>
      <th>p1</th>
      <th>p1_conf</th>
      <th>p1_dog</th>
      <th>p2</th>
      <th>p2_conf</th>
      <th>p2_dog</th>
      <th>p3</th>
      <th>p3_conf</th>
      <th>p3_dog</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>666020888022790149</td>
      <td>https://pbs.twimg.com/media/CT4udn0WwAA0aMy.jpg</td>
      <td>1</td>
      <td>Welsh_springer_spaniel</td>
      <td>0.465074</td>
      <td>True</td>
      <td>collie</td>
      <td>0.156665</td>
      <td>True</td>
      <td>Shetland_sheepdog</td>
      <td>0.061428</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>666029285002620928</td>
      <td>https://pbs.twimg.com/media/CT42GRgUYAA5iDo.jpg</td>
      <td>1</td>
      <td>redbone</td>
      <td>0.506826</td>
      <td>True</td>
      <td>miniature_pinscher</td>
      <td>0.074192</td>
      <td>True</td>
      <td>Rhodesian_ridgeback</td>
      <td>0.072010</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>666033412701032449</td>
      <td>https://pbs.twimg.com/media/CT4521TWwAEvMyu.jpg</td>
      <td>1</td>
      <td>German_shepherd</td>
      <td>0.596461</td>
      <td>True</td>
      <td>malinois</td>
      <td>0.138584</td>
      <td>True</td>
      <td>bloodhound</td>
      <td>0.116197</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>666044226329800704</td>
      <td>https://pbs.twimg.com/media/CT5Dr8HUEAA-lEu.jpg</td>
      <td>1</td>
      <td>Rhodesian_ridgeback</td>
      <td>0.408143</td>
      <td>True</td>
      <td>redbone</td>
      <td>0.360687</td>
      <td>True</td>
      <td>miniature_pinscher</td>
      <td>0.222752</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>666049248165822465</td>
      <td>https://pbs.twimg.com/media/CT5IQmsXIAAKY4A.jpg</td>
      <td>1</td>
      <td>miniature_pinscher</td>
      <td>0.560311</td>
      <td>True</td>
      <td>Rottweiler</td>
      <td>0.243682</td>
      <td>True</td>
      <td>Doberman</td>
      <td>0.154629</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python
image_predictions_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2075 entries, 0 to 2074
    Data columns (total 12 columns):
     #   Column    Non-Null Count  Dtype  
    ---  ------    --------------  -----  
     0   tweet_id  2075 non-null   int64  
     1   jpg_url   2075 non-null   object 
     2   img_num   2075 non-null   int64  
     3   p1        2075 non-null   object 
     4   p1_conf   2075 non-null   float64
     5   p1_dog    2075 non-null   bool   
     6   p2        2075 non-null   object 
     7   p2_conf   2075 non-null   float64
     8   p2_dog    2075 non-null   bool   
     9   p3        2075 non-null   object 
     10  p3_conf   2075 non-null   float64
     11  p3_dog    2075 non-null   bool   
    dtypes: bool(3), float64(3), int64(2), object(4)
    memory usage: 152.1+ KB
    


```python
list(image_predictions_df.columns)
```




    ['tweet_id',
     'jpg_url',
     'img_num',
     'p1',
     'p1_conf',
     'p1_dog',
     'p2',
     'p2_conf',
     'p2_dog',
     'p3',
     'p3_conf',
     'p3_dog']



# --------------------------------------------------------------------------------------------------------
# Quality Issues 

## archive_df

### - Converting timestamp column to date dtype
### - Converting tweet id from int to string
### - Removing the retweets and replies
### - Remove the unneeded columns (in_reply_to_status_id, in_reply_to_user_id, retweeted_status_id,	retweeted_status_user_id,	retweeted_status_timestamp) 
### - Removing href tags from Source column 
### - Removing missing values for expanded_urls 
### - Correcting the None values in stage
### - Replace possible wrong values and None with NaNs in name column
### - Correcting decimal numerators and column dtype
## api_df_now
### - Converting tweet_ids to str 
## image_predictions_df
### - Remove retweets from the df which was removed originally from archive_df
### - convert tweet ids to str
### - Predictions with False result in all predictions should be dropped.
# --------------------------------------------------------------------------------------------------------

# Tidiness Issues

## archive_df
### - Dog stage should be one column

## api_now_df

### - api_df should be a part of archived_df

## image_predictions_df
### - image_predictions_df should be a part of archive_df
# --------------------------------------------------------------------------------------------------------

# Cleaning Quality Issues

# archive_df


```python
#Creating a copy of the df
archive_clean_df = archive_df.copy()
```

## Define
### Converting timestamp column to date dtype
## Code


```python
#Convering the column with to_datetime method in pandas.
archive_clean_df.timestamp = pd.to_datetime(archive_df.timestamp)
```

## Define
### Removing missing values for expanded_urls
## Code


```python
# creating a list of tweet_ids with images "tweets_with_image" and confirming its length
tweets_with_image = list(image_predictions_df.tweet_id.unique())

# confirming that all the tweets with images exist in the archive dataset
len(tweets_with_image) == archive_clean_df.tweet_id.isin(tweets_with_image).sum()

# Cleaning in action
archive_clean_df = archive_clean_df[archive_clean_df.tweet_id.isin(tweets_with_image)]
```

## Define
### Convert tweet_id to string
## Code


```python
#Converting the tweet id column dtype to str as there isn't mathematical operations will be done on them.
archive_clean_df.tweet_id = archive_clean_df.tweet_id.astype(str)
```

## Define
### Removing the retweets and replies

## Code


```python
#Removing the retweets by excluding the notnull values. tilde (~) in the mask means not to include the following.
archive_clean_df = archive_clean_df[~((archive_clean_df['retweeted_status_id'].notnull()))]
#Removing the replies by excluding the notnull values. tilde (~) in the mask means not to include the following.
archive_clean_df = archive_clean_df[~((archive_clean_df['in_reply_to_status_id'].notnull()))]
```

## Define
### Removing Retweets and replies columns


```python
#Removing the retweets and replies columns with drop method in pandas.
archive_clean_df = archive_clean_df.drop(['in_reply_to_status_id', 'in_reply_to_user_id', 'retweeted_status_id', 
                                            'retweeted_status_user_id','retweeted_status_timestamp',], axis=1)
```

## Define
### Remove href tags from source column
## Code


```python
#importing BeatifulSoup library
from bs4 import BeautifulSoup as bs
source_list=[]
#Extracting all the values between the href tags and appending them to the list, source_list.
for row in archive_clean_df.source:
    source_href = bs(row, 'lxml')
    source_clean = source_href.find('a').contents[0]
    source_list.append(source_clean)
#Redifining the column source by adding the extracted values from href tags.
archive_clean_df['source'] = source_list
```

## Define
### Replace None values with empty strings in dog stages name
## Code


```python
#Replacing None values in dog stage columns with empty string with replace method.
archive_clean_df.iloc[:,-4:] = archive_clean_df.iloc[:, -4:].replace('None', '')
```

## Define
### Fixing Name column
## Code


```python
#Replacing the incorrect names with NaN values
archive_clean_df.loc[archive_clean_df.name == archive_clean_df.name.str.lower(), 'name'] = np.nan
```

## Define
### Correcting decimal numerators and column dtype
## Code


```python
#Converting the nonnull values in the rating_numerator column to floats by masking, indexing, and astype
archive_clean_df[archive_clean_df['rating_numerator'].notnull()]['rating_numerator'] = archive_clean_df[archive_clean_df['rating_numerator'].notnull()]['rating_numerator'].astype(float)
```


```python
#Correcting the decimal ratings by indexing
indexing = archive_clean_df[archive_clean_df.text.str.contains(r"(\d+\.\d*\/\d+)")][['text', 'rating_numerator']].index
archive_clean_df.loc[indexing[0], 'rating_numerator'] = 13.5
archive_clean_df.loc[indexing[1], 'rating_numerator'] = 9.75
archive_clean_df.loc[indexing[2], 'rating_numerator'] = 11.27
archive_clean_df.loc[indexing[3], 'rating_numerator'] = 11.26
#Incorrectly extracted rating as the regex ran over a number that was before the rating.
archive_clean_df.loc[2335, 'rating_numerator'] = 9
```

    C:\ProgramData\Anaconda3\lib\site-packages\pandas\core\strings.py:2001: UserWarning: This pattern has match groups. To actually get the groups, use str.extract.
      return func(self, *args, **kwargs)
    

# api_now_df



```python
api_clean = api_df_now.copy()
```

## Define
### Convert tweets_ids to str
## Code


```python
#Converting the tweet id column dtype to str as there isn't mathematical operations will be done on them.
api_clean.tweet_id = api_clean.tweet_id.astype(str)
```

# image_prediction_df


```python
image_clean = image_predictions_df.copy()
```

## Define
### Convert tweet ids to str
## Code


```python
image_clean.tweet_id = image_clean.tweet_id.astype(str)
```

## Define
### Drop retweets
## Code


```python
# Dropping the retweets and replies ids from the image prediction dataframe
image_clean = image_clean[~np.logical_not(image_clean.tweet_id.isin(list(archive_clean_df.tweet_id)))]
```

## Define
### Drop the values with all False predictions
## Code


```python
#Querying the predictions with False probabilities for each prediction.
df_false = image_clean.query('p1_dog == False & p2_dog == False & p3_dog == False')
#Dropping the mask df_false from the image predictions file.
image_clean.drop(df_false.index, axis=0, inplace=True)
```

# Cleaning Tidiness Issues

# archive_df

## Define
### Fix dog stage name as it should only have one column
## Code


```python
#Appending the dog stages in a one dog stage column.
archive_clean_df['dog_stage'] = archive_clean_df.iloc[:,-4] + archive_clean_df.iloc[:,-3] + archive_clean_df.iloc[:,-2] + archive_clean_df.iloc[:,-1]
```


```python
#Extracting the stage names from the strings concatenated.
archive_clean_df['dog_stage'] = archive_clean_df.dog_stage.str.extract(r'([\D]+)')
#replacing the stage names that has more than one name with the work "multiple"
archive_clean_df.replace({'dog_stage':{'doggopupper': 'multiple', 'doggopuppo':'multiple', 'doggofloofer':'multiple'}}, inplace=True)
```


```python
#Dropping the columns we appended.
archive_clean_df.drop(columns=['floofer','puppo','pupper','doggo'], axis=1, inplace=True)
```

# api_now_df

## Define
### Merging the data set with archive_clean_df as having two sets seems inconsistent
## Code


```python
#Merging the 2 datasets using merge method
archive_clean_df = archive_clean_df.merge(api_clean, how='left', on = 'tweet_id')
```

# image_predicitions_df
## Define
### df should be a part of archive_df
## Code


```python
#Merging the predictions file with the master data set and excluding the photo and number of photos columns
#as the master set has the urls for the photos
archive_clean_df = archive_clean_df.merge(image_clean[['tweet_id','p1','p1_conf','p1_dog','p2','p2_conf','p2_dog','p3','p3_conf','p3_dog']],
                                            how = 'left', on='tweet_id')
```


```python
#resetting the index to fix all the indexes differences that were made by adding or removing columns
archive_clean_df.reset_index(drop=True, inplace=True)
```


```python
#Saving the clean dataframe to a master dataset
archive_clean_df.to_csv('twitter_archive_master.csv', index=False)
```

# Visualizing the insights from the dataset


```python
#Loading the dataframes for visualization
archive_clean = pd.read_csv('twitter_archive_master.csv')
```


```python
#Converting the timestamp column to datetime dtype again
archive_clean.timestamp = pd.to_datetime(archive_clean.timestamp)
#Extracting a column only for analysis that includes year and month only
archive_clean['yearmonth'] = archive_clean.timestamp.dt.to_period('M')
```

    C:\ProgramData\Anaconda3\lib\site-packages\pandas\core\arrays\datetimes.py:1088: UserWarning: Converting to PeriodArray/Index representation will drop timezone information.
      warnings.warn(
    


```python
#Grouping by the time and getting the mean of the favorite count and retweet count to return a dataframe with the average
#of those by time
interactions_by_time = archive_clean.groupby('yearmonth')[['favorite_count', 'retweet_count']].mean()
```


```python
#Plotting the numbers to a line chart
interactions_by_time.plot.line(y= ['favorite_count', 'retweet_count'], figsize=(8,8))
plt.xlabel('Time', fontsize=15)
plt.ylabel('Retweets Counts Average', fontsize=13)
plt.show();
plt.savefig('Interactions vs time.png');
```
![Interactions vs time](https://user-images.githubusercontent.com/74722468/111870066-20484a00-898b-11eb-8ee6-b10bcc2c0961.png)




```python
#Getting the counts of the dog stages used and sorting them in ascended order.
stage_count = archive_clean.dog_stage.value_counts(ascending=True)
```


```python
#Plotting the count of the dog stage used.
stage_count.plot.bar(figsize=(8,11))
plt.xlabel('Dog Stage', fontsize=15)
plt.ylabel('Number Of Stages Used', fontsize=13)
plt.show();
plt.savefig('Dog Stage.png');
```

![Dog Stage](https://user-images.githubusercontent.com/74722468/111870048-03ac1200-898b-11eb-9e70-1c12f5b12fc7.png)



### It wasn't so easy to be done but it definitely was so fruitful experience. I hope you enjoyed going through this and enjoyed the insights.
