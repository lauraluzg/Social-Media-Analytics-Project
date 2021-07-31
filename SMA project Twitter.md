```python
import sys
!conda install -c conda-forge --yes --prefix {sys.prefix} tweepy wordcloud
```

    Collecting package metadata (current_repodata.json): done
    Solving environment: done
    
    # All requested packages already installed.
    



```python
import tweepy as tw
import config_twitter
```


```python
# establish Twitter API connection
auth = tw.OAuthHandler(config_twitter.consumer_key, config_twitter.consumer_secret)
auth.set_access_token(config_twitter.access_token, config_twitter.access_token_secret)
api = tw.API(auth, wait_on_rate_limit=True)
```


```python
# returns False if credentials could not be verified, otherwise a user-object
user = api.verify_credentials()
user
```




    User(_api=<tweepy.api.API object at 0x7f9099521b50>, _json={'id': 1414953849639612420, 'id_str': '1414953849639612420', 'name': 'Renata Perez', 'screen_name': 'RenataP56794649', 'location': '', 'description': '', 'url': None, 'entities': {'description': {'urls': []}}, 'protected': False, 'followers_count': 0, 'friends_count': 0, 'listed_count': 0, 'created_at': 'Tue Jul 13 14:24:36 +0000 2021', 'favourites_count': 0, 'utc_offset': None, 'time_zone': None, 'geo_enabled': False, 'verified': False, 'statuses_count': 0, 'lang': None, 'contributors_enabled': False, 'is_translator': False, 'is_translation_enabled': False, 'profile_background_color': 'F5F8FA', 'profile_background_image_url': None, 'profile_background_image_url_https': None, 'profile_background_tile': False, 'profile_image_url': 'http://abs.twimg.com/sticky/default_profile_images/default_profile_normal.png', 'profile_image_url_https': 'https://abs.twimg.com/sticky/default_profile_images/default_profile_normal.png', 'profile_link_color': '1DA1F2', 'profile_sidebar_border_color': 'C0DEED', 'profile_sidebar_fill_color': 'DDEEF6', 'profile_text_color': '333333', 'profile_use_background_image': True, 'has_extended_profile': True, 'default_profile': True, 'default_profile_image': True, 'following': False, 'follow_request_sent': False, 'notifications': False, 'translator_type': 'none', 'withheld_in_countries': [], 'suspended': False, 'needs_phone_verification': False}, id=1414953849639612420, id_str='1414953849639612420', name='Renata Perez', screen_name='RenataP56794649', location='', description='', url=None, entities={'description': {'urls': []}}, protected=False, followers_count=0, friends_count=0, listed_count=0, created_at=datetime.datetime(2021, 7, 13, 14, 24, 36), favourites_count=0, utc_offset=None, time_zone=None, geo_enabled=False, verified=False, statuses_count=0, lang=None, contributors_enabled=False, is_translator=False, is_translation_enabled=False, profile_background_color='F5F8FA', profile_background_image_url=None, profile_background_image_url_https=None, profile_background_tile=False, profile_image_url='http://abs.twimg.com/sticky/default_profile_images/default_profile_normal.png', profile_image_url_https='https://abs.twimg.com/sticky/default_profile_images/default_profile_normal.png', profile_link_color='1DA1F2', profile_sidebar_border_color='C0DEED', profile_sidebar_fill_color='DDEEF6', profile_text_color='333333', profile_use_background_image=True, has_extended_profile=True, default_profile=True, default_profile_image=True, following=False, follow_request_sent=False, notifications=False, translator_type='none', withheld_in_countries=[], suspended=False, needs_phone_verification=False)




```python
# Collect relevant tweets through the Twitter API.
import json
import tweepy as tw
```


```python
# IMPORTANT: enter proper access credential in config_twitter.py file
import config_twitter
```


```python
# function to establish an initial API connection, respecting the rate limit
def connect_api_client():
    auth = tw.OAuthHandler(config_twitter.consumer_key, config_twitter.consumer_secret)
    auth.set_access_token(config_twitter.access_token, config_twitter.access_token_secret)
    # https://docs.tweepy.org/en/stable/getting_started.html#api
    api = tw.API(auth, wait_on_rate_limit=True)
    try:
        # returns False if credentials could not be verified
        # https://docs.tweepy.org/en/stable/api.html#API.verify_credentials
        api.verify_credentials()
        user = api.verify_credentials()
        if not user:
            raise("Credentials could not be verified: Please check config.py")
        print(f"Connected to Twitter API as {user.name}")
    except Exception as e:
        raise e
    return api
```


```python
api = connect_api_client()
```

    Connected to Twitter API as Renata Perez



```python
# construct a search query
query = 'perfume OR "fragance" OR "women perfume" -filter:retweets'
```


```python
# decide how many tweets to query
ntweets = 2000
```


```python
# search and collect relevant tweets
tweets = [tweet._json for tweet in tw.Cursor(api.search, q=query, lang="en", tweet_mode='extended').items(ntweets)]
len(tweets)
```




    2000




```python
# example tweet content (json structure)
tweets[0]
```




    {'created_at': 'Fri Jul 30 21:00:26 +0000 2021',
     'id': 1421214123497508869,
     'id_str': '1421214123497508869',
     'full_text': '@alexander_olly @BBCR1 @anniemacmanus Perfume Genius 😁',
     'truncated': False,
     'display_text_range': [38, 54],
     'entities': {'hashtags': [],
      'symbols': [],
      'user_mentions': [{'screen_name': 'alexander_olly',
        'name': 'olly alexander',
        'id': 1345516880,
        'id_str': '1345516880',
        'indices': [0, 15]},
       {'screen_name': 'BBCR1',
        'name': 'BBC Radio 1',
        'id': 7111412,
        'id_str': '7111412',
        'indices': [16, 22]},
       {'screen_name': 'anniemacmanus',
        'name': 'Annie Mac',
        'id': 19811019,
        'id_str': '19811019',
        'indices': [23, 37]}],
      'urls': []},
     'metadata': {'iso_language_code': 'en', 'result_type': 'recent'},
     'source': '<a href="http://twitter.com/download/android" rel="nofollow">Twitter for Android</a>',
     'in_reply_to_status_id': 1420434384872747008,
     'in_reply_to_status_id_str': '1420434384872747008',
     'in_reply_to_user_id': 1345516880,
     'in_reply_to_user_id_str': '1345516880',
     'in_reply_to_screen_name': 'alexander_olly',
     'user': {'id': 1418321830666964993,
      'id_str': '1418321830666964993',
      'name': 'Mr R',
      'screen_name': 'MrR_Awhite',
      'location': 'West Midlands, England',
      'description': "New profile but still the same old 40 something gay. Probably watching Schitt's Creek, eating and moaning about weight gain. He/Him 🏳️\u200d🌈🏳️\u200d⚧️",
      'url': None,
      'entities': {'description': {'urls': []}},
      'protected': False,
      'followers_count': 128,
      'friends_count': 407,
      'listed_count': 0,
      'created_at': 'Thu Jul 22 21:27:42 +0000 2021',
      'favourites_count': 99,
      'utc_offset': None,
      'time_zone': None,
      'geo_enabled': False,
      'verified': False,
      'statuses_count': 51,
      'lang': None,
      'contributors_enabled': False,
      'is_translator': False,
      'is_translation_enabled': False,
      'profile_background_color': 'F5F8FA',
      'profile_background_image_url': None,
      'profile_background_image_url_https': None,
      'profile_background_tile': False,
      'profile_image_url': 'http://pbs.twimg.com/profile_images/1421200791260569617/0FblaDvY_normal.jpg',
      'profile_image_url_https': 'https://pbs.twimg.com/profile_images/1421200791260569617/0FblaDvY_normal.jpg',
      'profile_link_color': '1DA1F2',
      'profile_sidebar_border_color': 'C0DEED',
      'profile_sidebar_fill_color': 'DDEEF6',
      'profile_text_color': '333333',
      'profile_use_background_image': True,
      'has_extended_profile': True,
      'default_profile': True,
      'default_profile_image': False,
      'following': False,
      'follow_request_sent': False,
      'notifications': False,
      'translator_type': 'none',
      'withheld_in_countries': []},
     'geo': None,
     'coordinates': None,
     'place': None,
     'contributors': None,
     'is_quote_status': False,
     'retweet_count': 0,
     'favorite_count': 0,
     'favorited': False,
     'retweeted': False,
     'lang': 'en'}




```python
# save tweets data to json file
file_out = f"raw_tweet_data_{ntweets}.json"
with open(file_out, mode='w') as f:
    f.write(json.dumps(tweets, indent=2))
```


```python
# Twitter data analysis task starter.
import html
import json
import string
import re
from nltk import word_tokenize
from nltk.corpus import stopwords
from textblob import TextBlob
from wordcloud import WordCloud
import pandas as pd
import matplotlib.pyplot as plt
```


```python
# First collect the data in json-file; specify file name here (adjust the number as queried)
fjson = 'raw_tweet_data_2000.json'
```


```python
# read json file with tweets data
with open(fjson) as file:
    data = json.load(file)
len(data)
```




    2000




```python
# tweet data record example: as documented for the Twitter API
data[0]
```




    {'created_at': 'Fri Jul 30 21:00:26 +0000 2021',
     'id': 1421214123497508869,
     'id_str': '1421214123497508869',
     'full_text': '@alexander_olly @BBCR1 @anniemacmanus Perfume Genius 😁',
     'truncated': False,
     'display_text_range': [38, 54],
     'entities': {'hashtags': [],
      'symbols': [],
      'user_mentions': [{'screen_name': 'alexander_olly',
        'name': 'olly alexander',
        'id': 1345516880,
        'id_str': '1345516880',
        'indices': [0, 15]},
       {'screen_name': 'BBCR1',
        'name': 'BBC Radio 1',
        'id': 7111412,
        'id_str': '7111412',
        'indices': [16, 22]},
       {'screen_name': 'anniemacmanus',
        'name': 'Annie Mac',
        'id': 19811019,
        'id_str': '19811019',
        'indices': [23, 37]}],
      'urls': []},
     'metadata': {'iso_language_code': 'en', 'result_type': 'recent'},
     'source': '<a href="http://twitter.com/download/android" rel="nofollow">Twitter for Android</a>',
     'in_reply_to_status_id': 1420434384872747008,
     'in_reply_to_status_id_str': '1420434384872747008',
     'in_reply_to_user_id': 1345516880,
     'in_reply_to_user_id_str': '1345516880',
     'in_reply_to_screen_name': 'alexander_olly',
     'user': {'id': 1418321830666964993,
      'id_str': '1418321830666964993',
      'name': 'Mr R',
      'screen_name': 'MrR_Awhite',
      'location': 'West Midlands, England',
      'description': "New profile but still the same old 40 something gay. Probably watching Schitt's Creek, eating and moaning about weight gain. He/Him 🏳️\u200d🌈🏳️\u200d⚧️",
      'url': None,
      'entities': {'description': {'urls': []}},
      'protected': False,
      'followers_count': 128,
      'friends_count': 407,
      'listed_count': 0,
      'created_at': 'Thu Jul 22 21:27:42 +0000 2021',
      'favourites_count': 99,
      'utc_offset': None,
      'time_zone': None,
      'geo_enabled': False,
      'verified': False,
      'statuses_count': 51,
      'lang': None,
      'contributors_enabled': False,
      'is_translator': False,
      'is_translation_enabled': False,
      'profile_background_color': 'F5F8FA',
      'profile_background_image_url': None,
      'profile_background_image_url_https': None,
      'profile_background_tile': False,
      'profile_image_url': 'http://pbs.twimg.com/profile_images/1421200791260569617/0FblaDvY_normal.jpg',
      'profile_image_url_https': 'https://pbs.twimg.com/profile_images/1421200791260569617/0FblaDvY_normal.jpg',
      'profile_link_color': '1DA1F2',
      'profile_sidebar_border_color': 'C0DEED',
      'profile_sidebar_fill_color': 'DDEEF6',
      'profile_text_color': '333333',
      'profile_use_background_image': True,
      'has_extended_profile': True,
      'default_profile': True,
      'default_profile_image': False,
      'following': False,
      'follow_request_sent': False,
      'notifications': False,
      'translator_type': 'none',
      'withheld_in_countries': []},
     'geo': None,
     'coordinates': None,
     'place': None,
     'contributors': None,
     'is_quote_status': False,
     'retweet_count': 0,
     'favorite_count': 0,
     'favorited': False,
     'retweeted': False,
     'lang': 'en'}




```python
# create pandas dataframe from tweet text content
df_tweets = pd.DataFrame([t['full_text'] for t in data], columns=['text'])
df_tweets
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
      <th>text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>@alexander_olly @BBCR1 @anniemacmanus Perfume ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>This girl I’m working with must be stank cause...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>@g0thicd0lly literally any mens deodarant and ...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tl reminded me that i have bangchan's perfume\...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Enough to whisper from your lips\n I can smell...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>1995</th>
      <td>Playing right now - What&amp;amp;#039;s That Perfu...</td>
    </tr>
    <tr>
      <th>1996</th>
      <td>@nurinmunirahh Thank you!\n\nVICTORIA SECRET F...</td>
    </tr>
    <tr>
      <th>1997</th>
      <td>Got your hair in the wind and your skin gliste...</td>
    </tr>
    <tr>
      <th>1998</th>
      <td>ok but im still tryna comprehend how my mom js...</td>
    </tr>
    <tr>
      <th>1999</th>
      <td>@agudziunaite @bardzkreatywnie @toshiinotfound...</td>
    </tr>
  </tbody>
</table>
<p>2000 rows × 1 columns</p>
</div>




```python
# add selected columns from tweet data fields
df_tweets['retweets'] = [t['retweet_count'] for t in data]
df_tweets['favorites'] = [t['favorite_count'] for t in data]
df_tweets['user'] = [t['user']['screen_name'] for t in data]
df_tweets
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
      <th>text</th>
      <th>retweets</th>
      <th>favorites</th>
      <th>user</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>@alexander_olly @BBCR1 @anniemacmanus Perfume ...</td>
      <td>0</td>
      <td>0</td>
      <td>MrR_Awhite</td>
    </tr>
    <tr>
      <th>1</th>
      <td>This girl I’m working with must be stank cause...</td>
      <td>0</td>
      <td>0</td>
      <td>Nikkiis_nikki</td>
    </tr>
    <tr>
      <th>2</th>
      <td>@g0thicd0lly literally any mens deodarant and ...</td>
      <td>0</td>
      <td>0</td>
      <td>skinylegendmoth</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tl reminded me that i have bangchan's perfume\...</td>
      <td>0</td>
      <td>1</td>
      <td>bibiknows</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Enough to whisper from your lips\n I can smell...</td>
      <td>0</td>
      <td>0</td>
      <td>alibriz2</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1995</th>
      <td>Playing right now - What&amp;amp;#039;s That Perfu...</td>
      <td>0</td>
      <td>0</td>
      <td>WKNCHD2Playlist</td>
    </tr>
    <tr>
      <th>1996</th>
      <td>@nurinmunirahh Thank you!\n\nVICTORIA SECRET F...</td>
      <td>2</td>
      <td>0</td>
      <td>rumayyaa</td>
    </tr>
    <tr>
      <th>1997</th>
      <td>Got your hair in the wind and your skin gliste...</td>
      <td>0</td>
      <td>2</td>
      <td>DiamondMind86</td>
    </tr>
    <tr>
      <th>1998</th>
      <td>ok but im still tryna comprehend how my mom js...</td>
      <td>0</td>
      <td>0</td>
      <td>fffaraaah</td>
    </tr>
    <tr>
      <th>1999</th>
      <td>@agudziunaite @bardzkreatywnie @toshiinotfound...</td>
      <td>0</td>
      <td>1</td>
      <td>beviimo</td>
    </tr>
  </tbody>
</table>
<p>2000 rows × 4 columns</p>
</div>




```python
# text cleaning function: see prior class modules
stop_words = set(stopwords.words('english'))

# strictly speaking, this is a closure: uses a wider-scope variable stop_words
def text_cleanup(s):
    s_unesc = html.unescape(re.sub(r"http\S+", "", re.sub('\n+', ' ', s)))
    s_noemoji = s_unesc.encode('ascii', 'ignore').decode('ascii')
    # normalize to lowercase and tokenize
    wt = word_tokenize(s_noemoji.lower())
    
    # filter word-tokens
    wt_filt = [w for w in wt if (w not in stop_words) and (w not in string.punctuation) and (w.isalnum())]
    
    # return clean string
    return ' '.join(wt_filt)
```


```python
# add clean text column
df_tweets['text_clean'] = df_tweets['text'].apply(text_cleanup)
df_tweets
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
      <th>text</th>
      <th>retweets</th>
      <th>favorites</th>
      <th>user</th>
      <th>text_clean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>@alexander_olly @BBCR1 @anniemacmanus Perfume ...</td>
      <td>0</td>
      <td>0</td>
      <td>MrR_Awhite</td>
      <td>bbcr1 anniemacmanus perfume genius</td>
    </tr>
    <tr>
      <th>1</th>
      <td>This girl I’m working with must be stank cause...</td>
      <td>0</td>
      <td>0</td>
      <td>Nikkiis_nikki</td>
      <td>girl im working must stank cause sprayed perfu...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>@g0thicd0lly literally any mens deodarant and ...</td>
      <td>0</td>
      <td>0</td>
      <td>skinylegendmoth</td>
      <td>g0thicd0lly literally mens deodarant david bec...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tl reminded me that i have bangchan's perfume\...</td>
      <td>0</td>
      <td>1</td>
      <td>bibiknows</td>
      <td>tl reminded bangchan perfume brb need find fin...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Enough to whisper from your lips\n I can smell...</td>
      <td>0</td>
      <td>0</td>
      <td>alibriz2</td>
      <td>enough whisper lips smell perfume touch kind e...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1995</th>
      <td>Playing right now - What&amp;amp;#039;s That Perfu...</td>
      <td>0</td>
      <td>0</td>
      <td>WKNCHD2Playlist</td>
      <td>playing right 039 perfume wear jens lekman</td>
    </tr>
    <tr>
      <th>1996</th>
      <td>@nurinmunirahh Thank you!\n\nVICTORIA SECRET F...</td>
      <td>2</td>
      <td>0</td>
      <td>rumayyaa</td>
      <td>nurinmunirahh thank victoria secret fragrance ...</td>
    </tr>
    <tr>
      <th>1997</th>
      <td>Got your hair in the wind and your skin gliste...</td>
      <td>0</td>
      <td>2</td>
      <td>DiamondMind86</td>
      <td>got hair wind skin glistenin smell sweet perfu...</td>
    </tr>
    <tr>
      <th>1998</th>
      <td>ok but im still tryna comprehend how my mom js...</td>
      <td>0</td>
      <td>0</td>
      <td>fffaraaah</td>
      <td>ok im still tryna comprehend mom js decided us...</td>
    </tr>
    <tr>
      <th>1999</th>
      <td>@agudziunaite @bardzkreatywnie @toshiinotfound...</td>
      <td>0</td>
      <td>1</td>
      <td>beviimo</td>
      <td>agudziunaite bardzkreatywnie toshiinotfound ma...</td>
    </tr>
  </tbody>
</table>
<p>2000 rows × 5 columns</p>
</div>




```python
# sentiment analysis
def sentim_polarity(s):
    return TextBlob(s).sentiment.polarity

def sentim_subject(s):
    return TextBlob(s).sentiment.subjectivity

df_tweets['polarity'] = df_tweets['text_clean'].apply(sentim_polarity)
df_tweets['subjectivity'] = df_tweets['text_clean'].apply(sentim_subject)
df_tweets
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
      <th>text</th>
      <th>retweets</th>
      <th>favorites</th>
      <th>user</th>
      <th>text_clean</th>
      <th>polarity</th>
      <th>subjectivity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>@alexander_olly @BBCR1 @anniemacmanus Perfume ...</td>
      <td>0</td>
      <td>0</td>
      <td>MrR_Awhite</td>
      <td>bbcr1 anniemacmanus perfume genius</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>This girl I’m working with must be stank cause...</td>
      <td>0</td>
      <td>0</td>
      <td>Nikkiis_nikki</td>
      <td>girl im working must stank cause sprayed perfu...</td>
      <td>0.050000</td>
      <td>0.350000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>@g0thicd0lly literally any mens deodarant and ...</td>
      <td>0</td>
      <td>0</td>
      <td>skinylegendmoth</td>
      <td>g0thicd0lly literally mens deodarant david bec...</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tl reminded me that i have bangchan's perfume\...</td>
      <td>0</td>
      <td>1</td>
      <td>bibiknows</td>
      <td>tl reminded bangchan perfume brb need find fin...</td>
      <td>0.000000</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Enough to whisper from your lips\n I can smell...</td>
      <td>0</td>
      <td>0</td>
      <td>alibriz2</td>
      <td>enough whisper lips smell perfume touch kind e...</td>
      <td>0.200000</td>
      <td>0.633333</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1995</th>
      <td>Playing right now - What&amp;amp;#039;s That Perfu...</td>
      <td>0</td>
      <td>0</td>
      <td>WKNCHD2Playlist</td>
      <td>playing right 039 perfume wear jens lekman</td>
      <td>0.285714</td>
      <td>0.535714</td>
    </tr>
    <tr>
      <th>1996</th>
      <td>@nurinmunirahh Thank you!\n\nVICTORIA SECRET F...</td>
      <td>2</td>
      <td>0</td>
      <td>rumayyaa</td>
      <td>nurinmunirahh thank victoria secret fragrance ...</td>
      <td>0.095000</td>
      <td>0.630000</td>
    </tr>
    <tr>
      <th>1997</th>
      <td>Got your hair in the wind and your skin gliste...</td>
      <td>0</td>
      <td>2</td>
      <td>DiamondMind86</td>
      <td>got hair wind skin glistenin smell sweet perfu...</td>
      <td>0.425000</td>
      <td>0.575000</td>
    </tr>
    <tr>
      <th>1998</th>
      <td>ok but im still tryna comprehend how my mom js...</td>
      <td>0</td>
      <td>0</td>
      <td>fffaraaah</td>
      <td>ok im still tryna comprehend mom js decided us...</td>
      <td>0.233333</td>
      <td>0.250000</td>
    </tr>
    <tr>
      <th>1999</th>
      <td>@agudziunaite @bardzkreatywnie @toshiinotfound...</td>
      <td>0</td>
      <td>1</td>
      <td>beviimo</td>
      <td>agudziunaite bardzkreatywnie toshiinotfound ma...</td>
      <td>-0.354545</td>
      <td>0.718182</td>
    </tr>
  </tbody>
</table>
<p>2000 rows × 7 columns</p>
</div>




```python
# define the list of brands to analyze, consistent with the search topic
#  for which the tweets were collected
brands = ['marc jacobs', 'chanel', 'carolina herrera', 'givenchy', 'dior', 'burberry']
```


```python
# start a brand comparison dataframe
df_brands = pd.DataFrame(brands, columns=['brand'])
df_brands
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
      <th>brand</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>marc jacobs</td>
    </tr>
    <tr>
      <th>1</th>
      <td>chanel</td>
    </tr>
    <tr>
      <th>2</th>
      <td>carolina herrera</td>
    </tr>
    <tr>
      <th>3</th>
      <td>givenchy</td>
    </tr>
    <tr>
      <th>4</th>
      <td>dior</td>
    </tr>
    <tr>
      <th>5</th>
      <td>burberry</td>
    </tr>
  </tbody>
</table>
</div>




```python
# example: tweet subset mentioning a given brand
df_tweets[df_tweets['text_clean'].str.contains("marc jacobs")]
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
      <th>text</th>
      <th>retweets</th>
      <th>favorites</th>
      <th>user</th>
      <th>text_clean</th>
      <th>polarity</th>
      <th>subjectivity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>564</th>
      <td>All your favourite designer fragrances\nunder ...</td>
      <td>0</td>
      <td>0</td>
      <td>Rightdose_UK</td>
      <td>favourite designer fragrances one roof deliver...</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>606</th>
      <td>@g0thicd0lly no specific scent but actual perf...</td>
      <td>0</td>
      <td>0</td>
      <td>melancholym0th</td>
      <td>g0thicd0lly specific scent actual perfume dais...</td>
      <td>0.000000</td>
      <td>0.112500</td>
    </tr>
    <tr>
      <th>742</th>
      <td>Must Read: StockX Is Getting Into Beauty, Marc...</td>
      <td>0</td>
      <td>0</td>
      <td>ATMBoutique</td>
      <td>must read stockx getting beauty marc jacobs ne...</td>
      <td>0.136364</td>
      <td>0.454545</td>
    </tr>
    <tr>
      <th>1120</th>
      <td>I just want the new marc jacobs, and Chanel &amp;a...</td>
      <td>0</td>
      <td>1</td>
      <td>ChynnaMekyala</td>
      <td>want new marc jacobs chanel im done buying per...</td>
      <td>0.136364</td>
      <td>0.454545</td>
    </tr>
    <tr>
      <th>1524</th>
      <td>Marc jacobs decadence (perfume gift set) .. \n...</td>
      <td>2</td>
      <td>3</td>
      <td>Rukee_xo</td>
      <td>marc jacobs decadence perfume gift set n10k na...</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_tweets['text_clean'][564]
```




    'favourite designer fragrances one roof delivered door calvin klein dolce gabbana marc jacobs jimmy choo hugo prada gucci burberry shop today aftershave perfume cologne smellgood feelgood rightdose'




```python
df_tweets['text'][742]
```




    'Must Read: StockX Is Getting Into Beauty, Marc Jacobs Has a New Perfume https://t.co/XCV9kBbtse https://t.co/JGdt6TnlpP'




```python
df_tweets['text_clean'][1120]
```




    'want new marc jacobs chanel im done buying perfume till christmas'




```python
# function to compute average sentiment of tweets mentioning a given brand
def brand_sentiment(b):
    return df_tweets[df_tweets['text_clean'].str.contains(b)]['polarity'].mean()
```


```python
# brand sentiment comparison
df_brands['average_sentiment'] = df_brands['brand'].apply(brand_sentiment)
df_brands
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
      <th>brand</th>
      <th>average_sentiment</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>marc jacobs</td>
      <td>0.054545</td>
    </tr>
    <tr>
      <th>1</th>
      <td>chanel</td>
      <td>0.097359</td>
    </tr>
    <tr>
      <th>2</th>
      <td>carolina herrera</td>
      <td>0.224053</td>
    </tr>
    <tr>
      <th>3</th>
      <td>givenchy</td>
      <td>-0.052381</td>
    </tr>
    <tr>
      <th>4</th>
      <td>dior</td>
      <td>0.071481</td>
    </tr>
    <tr>
      <th>5</th>
      <td>burberry</td>
      <td>0.200000</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_tweets.sort_values(by='polarity', ascending=False).head(10)
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
      <th>text</th>
      <th>retweets</th>
      <th>favorites</th>
      <th>user</th>
      <th>text_clean</th>
      <th>polarity</th>
      <th>subjectivity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1660</th>
      <td>@swagc0untry use kobra perfume, best</td>
      <td>0</td>
      <td>0</td>
      <td>sanyxm</td>
      <td>swagc0untry use kobra perfume best</td>
      <td>1.0</td>
      <td>0.3</td>
    </tr>
    <tr>
      <th>1369</th>
      <td>#CanYaman released a perfume, all perfumes are...</td>
      <td>1</td>
      <td>12</td>
      <td>Meche1069</td>
      <td>canyaman released perfume perfumes sold awesome</td>
      <td>1.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>93</th>
      <td>Tell me the name of the best perfume you’ve ev...</td>
      <td>0</td>
      <td>10</td>
      <td>w1tvh</td>
      <td>tell name best perfume youve ever used</td>
      <td>1.0</td>
      <td>0.3</td>
    </tr>
    <tr>
      <th>678</th>
      <td>@DrJessTaylor Best ones for me are the cheopo ...</td>
      <td>0</td>
      <td>0</td>
      <td>DaphneBroon2</td>
      <td>drjesstaylor best ones cheopo corner shop ones...</td>
      <td>1.0</td>
      <td>0.3</td>
    </tr>
    <tr>
      <th>324</th>
      <td>YSL perfume is the best.</td>
      <td>0</td>
      <td>0</td>
      <td>YeahImCicii_</td>
      <td>ysl perfume best</td>
      <td>1.0</td>
      <td>0.3</td>
    </tr>
    <tr>
      <th>1424</th>
      <td>25 Best Selling Perfumes For Women #checkitout...</td>
      <td>0</td>
      <td>0</td>
      <td>widestcom</td>
      <td>25 best selling perfumes women checkitout cool...</td>
      <td>1.0</td>
      <td>0.3</td>
    </tr>
    <tr>
      <th>198</th>
      <td>Not me spending $140 on a perfume that smells ...</td>
      <td>0</td>
      <td>0</td>
      <td>Fonzy_Fonseca</td>
      <td>spending 140 perfume smells delicious</td>
      <td>1.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>799</th>
      <td>$WNBD Hello Kent - Yes, the advancement of Nia...</td>
      <td>2</td>
      <td>7</td>
      <td>WinningCEO</td>
      <td>wnbd hello kent yes advancement niagara mist p...</td>
      <td>1.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>961</th>
      <td>hi best friends. whats yalls fave perfume 😭</td>
      <td>0</td>
      <td>0</td>
      <td>bblanchettoroni</td>
      <td>hi best friends whats yalls fave perfume</td>
      <td>1.0</td>
      <td>0.3</td>
    </tr>
    <tr>
      <th>964</th>
      <td>The best perfume 😄 https://t.co/5U6M3n3mrp</td>
      <td>1</td>
      <td>8</td>
      <td>aheartforhoran</td>
      <td>best perfume</td>
      <td>1.0</td>
      <td>0.3</td>
    </tr>
  </tbody>
</table>
</div>




```python
# most retweeted content
df_tweets.sort_values(by='retweets', ascending=False).head(10)
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
      <th>text</th>
      <th>retweets</th>
      <th>favorites</th>
      <th>user</th>
      <th>text_clean</th>
      <th>polarity</th>
      <th>subjectivity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1589</th>
      <td>Perfume’s first EP “Polygon Wave EP” confirmed...</td>
      <td>494</td>
      <td>1913</td>
      <td>perfumeofficial</td>
      <td>perfumes first ep polygon wave ep confirmed re...</td>
      <td>0.325000</td>
      <td>0.666667</td>
    </tr>
    <tr>
      <th>874</th>
      <td>That Santa Maria perfume gonna be done in a we...</td>
      <td>117</td>
      <td>284</td>
      <td>CuffinTen</td>
      <td>santa maria perfume gon na done week bless</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>1295</th>
      <td>🇪🇸📸| #CanYaman’s photos for launch of his new ...</td>
      <td>66</td>
      <td>135</td>
      <td>CanYamanMedia</td>
      <td>canyamans photos launch new perfume canyamanma...</td>
      <td>0.136364</td>
      <td>0.454545</td>
    </tr>
    <tr>
      <th>911</th>
      <td>Perfume and cats been his brand https://t.co/v...</td>
      <td>66</td>
      <td>111</td>
      <td>CuffinTen</td>
      <td>perfume cats brand</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>765</th>
      <td>idc what anybody say i’m spraying myself with ...</td>
      <td>49</td>
      <td>175</td>
      <td>xelessence</td>
      <td>idc anybody say im spraying perfume go bed hab...</td>
      <td>0.300000</td>
      <td>0.550000</td>
    </tr>
    <tr>
      <th>1796</th>
      <td>Egyptian perfume bottle with an Amarna princes...</td>
      <td>47</td>
      <td>247</td>
      <td>_AcrossTheAges</td>
      <td>egyptian perfume bottle amarna princess standi...</td>
      <td>0.300000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>1844</th>
      <td>don't forget to get your sunsilk natural perfu...</td>
      <td>40</td>
      <td>21</td>
      <td>M4Metawin</td>
      <td>forget get sunsilk natural perfume blossom ava...</td>
      <td>0.300000</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>1334</th>
      <td>May be you should drink some perfume, so you c...</td>
      <td>37</td>
      <td>116</td>
      <td>vkrutika</td>
      <td>may drink perfume smell good inside</td>
      <td>0.700000</td>
      <td>0.600000</td>
    </tr>
    <tr>
      <th>410</th>
      <td>I was reading a thread in which someone was lo...</td>
      <td>36</td>
      <td>210</td>
      <td>jdesmondharris</td>
      <td>reading thread someone looking dupe perfume so...</td>
      <td>0.162500</td>
      <td>0.713889</td>
    </tr>
    <tr>
      <th>860</th>
      <td>Ten be enjoying not just a customized perfume ...</td>
      <td>28</td>
      <td>19</td>
      <td>J10Ncity</td>
      <td>ten enjoying customized perfume whole set sham...</td>
      <td>0.285714</td>
      <td>0.407143</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_tweets['text'][1334]
```




    'May be you should drink some perfume, so you can smell good on the inside too!'




```python
df_tweets['text'][410]
```




    'I was reading a thread in which someone was looking for a dupe for a perfume and someone replied "I use X and I\'m not sure because I had covid and can\'t smell but people tell me it\'s nice." People are just casually dealing with covid stuff that we barely talk about'




```python
# users producing most retweeted content
df_tweets.sort_values(by='retweets', ascending=False).head(10)['user']
```




    1589    perfumeofficial
    874           CuffinTen
    1295      CanYamanMedia
    911           CuffinTen
    765          xelessence
    1796     _AcrossTheAges
    1844          M4Metawin
    1334           vkrutika
    410      jdesmondharris
    860            J10Ncity
    Name: user, dtype: object




```python
# combine all text for a specific brand
def brand_all_text(b):
    # https://stackoverflow.com/a/51871650
    return ' '.join(df_tweets[df_tweets['text_clean'].str.contains(b)]['text_clean'])
```


```python
# most common twet content keywords for a specific brand
b = 'marc jacobs'
wc = WordCloud(width=1200, height=800, max_font_size=110, collocations=False).generate(brand_all_text(b))
plt.axis("off")
plt.imshow(wc, interpolation="bilinear")
plt.show()
```


    
![png](output_36_0.png)
    



```python
# example: tweet subset mentioning a given brand
df_tweets[df_tweets['text_clean'].str.contains("chanel")]
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
      <th>text</th>
      <th>retweets</th>
      <th>favorites</th>
      <th>user</th>
      <th>text_clean</th>
      <th>polarity</th>
      <th>subjectivity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>67</th>
      <td>@nickamin20 I think 1970s. Stunning, sophistic...</td>
      <td>0</td>
      <td>0</td>
      <td>AdrienneMH2425</td>
      <td>nickamin20 think 1970s stunning sophisticated ...</td>
      <td>0.333333</td>
      <td>0.666667</td>
    </tr>
    <tr>
      <th>107</th>
      <td>What’s a good ass perfume that attracts the fe...</td>
      <td>0</td>
      <td>0</td>
      <td>_taylor305</td>
      <td>whats good ass perfume attracts fellas besides...</td>
      <td>0.700000</td>
      <td>0.600000</td>
    </tr>
    <tr>
      <th>158</th>
      <td>Men's Perfume Allure Homme Ed.Blanche Chanel E...</td>
      <td>0</td>
      <td>0</td>
      <td>UniqPerfumes</td>
      <td>men perfume allure homme chanel edp 150 ml</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>538</th>
      <td>@prodbangtan__ @xiumintiddies I’m so jealous a...</td>
      <td>0</td>
      <td>0</td>
      <td>lololol24611121</td>
      <td>xiumintiddies im jealous asf smell smell hmm m...</td>
      <td>0.200000</td>
      <td>0.250000</td>
    </tr>
    <tr>
      <th>621</th>
      <td>I get more compliments when I wear my cloud pe...</td>
      <td>0</td>
      <td>0</td>
      <td>KarinaM32</td>
      <td>get compliments wear cloud perfume wear chanel...</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>798</th>
      <td>I use my Chanel perfume to go to bed cause its...</td>
      <td>0</td>
      <td>0</td>
      <td>glencocooooo_</td>
      <td>use chanel perfume go bed cause soft also chan...</td>
      <td>0.133333</td>
      <td>0.583333</td>
    </tr>
    <tr>
      <th>953</th>
      <td>Louis bag, MCM bag, mini Chanel, and dior sadd...</td>
      <td>0</td>
      <td>0</td>
      <td>M4_Mazhae</td>
      <td>louis bag mcm bag mini chanel dior saddle look...</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>958</th>
      <td>CHANEL (the perfume) Manufactured "𝘚𝘔𝘈𝘙𝘛 𝘎𝘜𝘐𝘓𝘓...</td>
      <td>0</td>
      <td>0</td>
      <td>Talia_1976</td>
      <td>chanel perfume manufactured china shipped fema...</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>1120</th>
      <td>I just want the new marc jacobs, and Chanel &amp;a...</td>
      <td>0</td>
      <td>1</td>
      <td>ChynnaMekyala</td>
      <td>want new marc jacobs chanel im done buying per...</td>
      <td>0.136364</td>
      <td>0.454545</td>
    </tr>
    <tr>
      <th>1207</th>
      <td>BUNDLE OF THE WEEK!😎\n\n1 RM40\n2 RM60\n3 RM80...</td>
      <td>2</td>
      <td>2</td>
      <td>stixmaperfume</td>
      <td>bundle week 1 rm40 2 rm60 3 rm80 promo untuk 1...</td>
      <td>0.000000</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <th>1537</th>
      <td>This Chanel perfume gone get them everytime 😌😜</td>
      <td>0</td>
      <td>0</td>
      <td>teadaddyy</td>
      <td>chanel perfume gone get everytime</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>1559</th>
      <td>HIGH JEWELRY N°5 COLLECTION\nThe Grasse Jasmin...</td>
      <td>0</td>
      <td>0</td>
      <td>SinimaleFreddy</td>
      <td>high jewelry n5 collection grasse jasmine ring...</td>
      <td>0.160000</td>
      <td>0.540000</td>
    </tr>
    <tr>
      <th>1580</th>
      <td>My ex might hate me but she taught me so much ...</td>
      <td>0</td>
      <td>1</td>
      <td>shelliesnout</td>
      <td>ex might hate taught much quality makeup perfu...</td>
      <td>-0.300000</td>
      <td>0.550000</td>
    </tr>
    <tr>
      <th>1648</th>
      <td>Getting me my Chanel perfume 🥰</td>
      <td>0</td>
      <td>0</td>
      <td>odalysa_x3</td>
      <td>getting chanel perfume</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_tweets['text'][1537]
```




    'This Chanel perfume gone get them everytime 😌😜'




```python
df_tweets['text'][621]
```




    'I get more compliments when I wear my cloud perfume than when I wear my Chanel and YSL'




```python
# combine all text for a specific brand
def brand_all_text(b):
    # https://stackoverflow.com/a/51871650
    return ' '.join(df_tweets[df_tweets['text_clean'].str.contains(b)]['text_clean'])
```


```python
# most common twet content keywords for a specific brand
b = 'chanel'
wc = WordCloud(width=1200, height=800, max_font_size=110, collocations=False).generate(brand_all_text(b))
plt.axis("off")
plt.imshow(wc, interpolation="bilinear")
plt.show()
```


    
![png](output_41_0.png)
    



```python
# example: tweet subset mentioning a given brand
df_tweets[df_tweets['text_clean'].str.contains("carolina herrera")]
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
      <th>text</th>
      <th>retweets</th>
      <th>favorites</th>
      <th>user</th>
      <th>text_clean</th>
      <th>polarity</th>
      <th>subjectivity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>426</th>
      <td>some of u bitches made fun of me bc i was in a...</td>
      <td>0</td>
      <td>0</td>
      <td>kissyancy</td>
      <td>u bitches made fun bc summer camp coding karli...</td>
      <td>0.218182</td>
      <td>0.327273</td>
    </tr>
    <tr>
      <th>1011</th>
      <td>My favorite thing about the Carolina Herrera p...</td>
      <td>0</td>
      <td>0</td>
      <td>Iam_bnice</td>
      <td>favorite thing carolina herrera perfume scents...</td>
      <td>0.566667</td>
      <td>0.933333</td>
    </tr>
    <tr>
      <th>1500</th>
      <td>I really can’t figure out the hype behind this...</td>
      <td>0</td>
      <td>0</td>
      <td>xNewYork_Times</td>
      <td>really cant figure hype behind carolina herrer...</td>
      <td>-0.025000</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <th>1584</th>
      <td>There’s a new Carolina Herrera perfume and no ...</td>
      <td>0</td>
      <td>0</td>
      <td>Iam_bnice</td>
      <td>theres new carolina herrera perfume one told</td>
      <td>0.136364</td>
      <td>0.454545</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_tweets['text'][1011]
```




    'My favorite thing about the Carolina Herrera perfume scents is they’re amazing on their own and they work amazingly with other scents'




```python
df_tweets['text'][1500]
```




    'I really can’t figure out the hype behind this Carolina Herrera “Good Girl” perfume. It stinks 🤢'




```python
# combine all text for a specific brand
def brand_all_text(b):
    # https://stackoverflow.com/a/51871650
    return ' '.join(df_tweets[df_tweets['text_clean'].str.contains(b)]['text_clean'])
```


```python
# most common twet content keywords for a specific brand
b = 'carolina herrera'
wc = WordCloud(width=1200, height=800, max_font_size=110, collocations=False).generate(brand_all_text(b))
plt.axis("off")
plt.imshow(wc, interpolation="bilinear")
plt.show()
```


    
![png](output_46_0.png)
    



```python
# example: tweet subset mentioning a given brand
df_tweets[df_tweets['text_clean'].str.contains("givenchy")]
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
      <th>text</th>
      <th>retweets</th>
      <th>favorites</th>
      <th>user</th>
      <th>text_clean</th>
      <th>polarity</th>
      <th>subjectivity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1046</th>
      <td>@_BayBey I dead ass just gave my coworker some...</td>
      <td>0</td>
      <td>0</td>
      <td>_shortyb</td>
      <td>dead ass gave coworker givenchy perfume bought...</td>
      <td>-0.200000</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>1166</th>
      <td>@Joannechocolat Oooh I may have to try it. I’m...</td>
      <td>0</td>
      <td>0</td>
      <td>SnifferOfBooks_</td>
      <td>joannechocolat oooh may try im obsessed givenc...</td>
      <td>0.095238</td>
      <td>0.811905</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_tweets['text'][1046]
```




    '@_BayBey I dead ass just gave my coworker some Givenchy perfume that I bought 2 years ago and only sprayed twice 😬'




```python
df_tweets['text'][1166]
```




    '@Joannechocolat Oooh I may have to try it. I’m obsessed with Givenchy’s latest perfume but I can never pronounce it right'




```python
# combine all text for a specific brand
def brand_all_text(b):
    # https://stackoverflow.com/a/51871650
    return ' '.join(df_tweets[df_tweets['text_clean'].str.contains(b)]['text_clean'])
```


```python
# most common twet content keywords for a specific brand
b = 'givenchy'
wc = WordCloud(width=1200, height=800, max_font_size=110, collocations=False).generate(brand_all_text(b))
plt.axis("off")
plt.imshow(wc, interpolation="bilinear")
plt.show()
```


    
![png](output_51_0.png)
    



```python
# example: tweet subset mentioning a given brand
df_tweets[df_tweets['text_clean'].str.contains("dior")]
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
      <th>text</th>
      <th>retweets</th>
      <th>favorites</th>
      <th>user</th>
      <th>text_clean</th>
      <th>polarity</th>
      <th>subjectivity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>67</th>
      <td>@nickamin20 I think 1970s. Stunning, sophistic...</td>
      <td>0</td>
      <td>0</td>
      <td>AdrienneMH2425</td>
      <td>nickamin20 think 1970s stunning sophisticated ...</td>
      <td>0.333333</td>
      <td>0.666667</td>
    </tr>
    <tr>
      <th>325</th>
      <td>Shall I compare thee to the title page?\nThou ...</td>
      <td>0</td>
      <td>0</td>
      <td>SONNETS_ON_NET</td>
      <td>shall compare thee title page thou art cuddly ...</td>
      <td>-0.050000</td>
      <td>0.350000</td>
    </tr>
    <tr>
      <th>953</th>
      <td>Louis bag, MCM bag, mini Chanel, and dior sadd...</td>
      <td>0</td>
      <td>0</td>
      <td>M4_Mazhae</td>
      <td>louis bag mcm bag mini chanel dior saddle look...</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>1202</th>
      <td>Elizabeth - Miss Dior - Bau fruity floral and ...</td>
      <td>0</td>
      <td>0</td>
      <td>stixmaperfume</td>
      <td>elizabeth miss dior bau fruity floral pink pep...</td>
      <td>0.420000</td>
      <td>0.710000</td>
    </tr>
    <tr>
      <th>1209</th>
      <td>@Zama_Omuhle @leahmadib They don't have this D...</td>
      <td>0</td>
      <td>0</td>
      <td>Glamology_ZA</td>
      <td>leahmadib dior think exclusive dior store anyw...</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>1284</th>
      <td>@RU3YJANE miss dior perfume</td>
      <td>0</td>
      <td>0</td>
      <td>mfintii</td>
      <td>ru3yjane miss dior perfume</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>1511</th>
      <td>I must cop the dior sauvage fragance</td>
      <td>0</td>
      <td>0</td>
      <td>IncognitoJov</td>
      <td>must cop dior sauvage fragance</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>1536</th>
      <td>Shall I compare thee to LL Cool J?\nThou art m...</td>
      <td>0</td>
      <td>0</td>
      <td>SONNETS_ON_NET</td>
      <td>shall compare thee cool j thou art clumsy unaf...</td>
      <td>-0.010000</td>
      <td>0.550000</td>
    </tr>
    <tr>
      <th>1805</th>
      <td>Shall I compare thee to Cartesian plane?\nThou...</td>
      <td>0</td>
      <td>0</td>
      <td>SONNETS_ON_NET</td>
      <td>shall compare thee cartesian plane thou art th...</td>
      <td>-0.050000</td>
      <td>0.233333</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_tweets['text'][1202]
```




    'Elizabeth - Miss Dior - Bau fruity floral and pink pepper! Very beautiful, elegant &amp; feminine. Siapa yang suka sweet fruity floral type of perfume, you will love this one. And also ada hint of caramel dalam perfume ni.'




```python
# combine all text for a specific brand
def brand_all_text(b):
    # https://stackoverflow.com/a/51871650
    return ' '.join(df_tweets[df_tweets['text_clean'].str.contains(b)]['text_clean'])
```


```python
# most common twet content keywords for a specific brand
b = 'dior'
wc = WordCloud(width=1200, height=800, max_font_size=110, collocations=False).generate(brand_all_text(b))
plt.axis("off")
plt.imshow(wc, interpolation="bilinear")
plt.show()
```


    
![png](output_55_0.png)
    



```python
# example: tweet subset mentioning a given brand
df_tweets[df_tweets['text_clean'].str.contains("burberry")]
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
      <th>text</th>
      <th>retweets</th>
      <th>favorites</th>
      <th>user</th>
      <th>text_clean</th>
      <th>polarity</th>
      <th>subjectivity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>162</th>
      <td>@burberry #weekend #burberry \nLove this perfu...</td>
      <td>0</td>
      <td>0</td>
      <td>LoveDiamonds09</td>
      <td>burberry weekend burberry love perfume smells ...</td>
      <td>0.55</td>
      <td>0.75</td>
    </tr>
    <tr>
      <th>225</th>
      <td>Mini fridge\nNaked eyeshadow pallet\nGucci per...</td>
      <td>0</td>
      <td>0</td>
      <td>lilliana_mojica</td>
      <td>mini fridge naked eyeshadow pallet gucci perfu...</td>
      <td>0.25</td>
      <td>0.50</td>
    </tr>
    <tr>
      <th>564</th>
      <td>All your favourite designer fragrances\nunder ...</td>
      <td>0</td>
      <td>0</td>
      <td>Rightdose_UK</td>
      <td>favourite designer fragrances one roof deliver...</td>
      <td>0.00</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>1579</th>
      <td>Adam Driver’s Burberry Fragrance Ad Set the In...</td>
      <td>3</td>
      <td>8</td>
      <td>CelebMagTM</td>
      <td>adam drivers burberry fragrance ad set interne...</td>
      <td>0.00</td>
      <td>0.00</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_tweets['text'][162]
```




    '@burberry #weekend #burberry \nLove this perfume smells amazing it’s one of my faves ❤️ #eaudeparfum https://t.co/zDciwKlGcx'




```python
# combine all text for a specific brand
def brand_all_text(b):
    # https://stackoverflow.com/a/51871650
    return ' '.join(df_tweets[df_tweets['text_clean'].str.contains(b)]['text_clean'])
```


```python
# most common twet content keywords for a specific brand
b = 'burberry'
wc = WordCloud(width=1200, height=800, max_font_size=110, collocations=False).generate(brand_all_text(b))
plt.axis("off")
plt.imshow(wc, interpolation="bilinear")
plt.show()
```


    
![png](output_59_0.png)
    



```python

```
