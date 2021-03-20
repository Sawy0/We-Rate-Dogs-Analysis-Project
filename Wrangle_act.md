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


```python
#Listing the download links to all the requried documents for the project.
links = ['https://video.udacity-data.com/topher/2018/November/5bf60fbf_twitter-archive-enhanced/twitter-archive-enhanced.csv',
        'https://video.udacity-data.com/topher/2018/November/5bf60fda_tweet-json/tweet-json',
        'https://video.udacity-data.com/topher/2018/November/5bf60fe7_image-predictions/image-predictions.tsv',
        'https://video.udacity-data.com/topher/2018/November/5bfc35b7_twitter-api/twitter-api.rtf',
        'https://video.udacity-data.com/topher/2018/November/5be5fb4c_twitter-api/twitter-api.py']
```


```python
#Getting the requests from the above links and writing them to the workspace directory.
for link in links:
    response = requests.get(link)
    with open(os.path.join('/home/workspace',link.split("/")[-1]), mode = 'wb') as file:
        file.write(response.content)
```


```python
archive_df = pd.read_csv("twitter-archive-enhanced.csv")
image_predictions_df = pd.read_csv("image-predictions.tsv", sep='\t')
```


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


```python
#Making a DataFrame from the tweets gathered in the json file.
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


    <IPython.core.display.Javascript object>



<img src="data:image/png;base64,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" width="576">



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


    <IPython.core.display.Javascript object>



<img src="data:image/png;base64,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" width="576">



```python

```
