# Reddit Political Subreddit Comments during the 2016 U.S. Presidential Election

View the presentation for this project [here](http://bit.ly/2JbFA1S).

## Data Acquisition and Cleaning


#### Acquisition

We downloaded all Reddit comments during the months of October (RC_2016-10.bz2) and November (RC_2016-11.bz2) 2016 from [pushshift](http://files.pushshift.io/reddit/comments/). However, comments from the months of _December 2005_ to _February 2018_ ranging from 118K bytes to 7.75B bytes. The compressed files contain a collection of JSON objects representing reddit comments. Here is an example of the JSON object: 
 

```{JSON}
{  
    "author":"Dethcola",
    "author_flair_css_class":"",
    "author_flair_text":"Clairemont",
    "body":"A quarry",
    "can_gild":true,
    "controversiality":0,
    "created_utc":1506816000,
    "distinguished":null,
    "edited":false,
    "gilded":0,
    "id":"dnqik14",
    "is_submitter":false,
    "link_id":"t3_73ieyz",
    "parent_id":"t3_73ieyz",
    "permalink":"/r/sandiego/comments/73ieyz/best_place_for_granite_counter_tops/dnqik14/",
    "retrieved_on":1509189606,
    "score":3,
    "stickied":false,
    "subreddit":"sandiego",
    "subreddit_id":"t5_2qq2q"
}
```

The **October** dataset was 6.45 GBs compressed and ~35.2 GBs uncompressed for a total of 54,129,644 JSON objects.  The **November** dataset was 6.45 GBs compressed and ~46.71 GBs uncompressed for a total of 71,826,554 JSON objects. 

#### Cleaning

The cleaning of the downloaded data sets include multiple phases due to the magnitude of the data sets. The first phase of filtering is meant to decrease the size of the data set by selecting comments from 2 weeks before election day (October 26) to 2 weeks after election day (November 23). Additionally, we only keep comments that were posted in the list of [political subreddits](https://github.com/jShiohaha/redditCommentsAndPresidentialElection/blob/master/Data/PoliticalSubreddits.csv).

Additional cleaning and filtering of the data set is described below:

- Removed comments if the author was one of the following - 
  - [deleted]
  - AutoModerator
  - Contains the word `bot`
- Remove comments if the author was a [known moderator](https://files.pushshift.io/reddit/moderators/) due to the fact that the majority of their comments were relating to their moderator dutes. This removal of mods from the list of known moderators removed 233 comments.
- Comments were not in the [list of relevant subreddits](https://raw.githubusercontent.com/jShiohaha/redditCommentsAndPresidentialElection/master/Data/PoliticalSubreddits.csv).
- Remove all hardcoded white space characters in the body of comment text, such as `\n\r\t`
- Features were selectively kept based on the data required to do the analysis in this project.

```{JSON}
{  
    "author":"Dethcola",
    "body":"A quarry",
    "created_utc":1506816000,
    "score":3,
    "subreddit":"sandiego",
    "subreddit_id":"t5_2qq2q"
}
```

#### Partitioning

Dates around the election, broken up into varying sizes. **Note**: All UTC stamps for dates were calculated from the date at 0:0:0 in CT. For example, the range of October 26 to November 1 would be October 26 at 0:0:0 to November 2 at 0:0:0, so we consider all comments on nov 1. Initially done with [Epoch Converter](https://www.epochconverter.com), but later wrote time conversion functions.

## Sentiment Analysis

**What is Sentiment Analysis?**

Sentiment analysis, or opinion mining, is an active area of
study in the field of natural language processing that analyzes people's opinions, sentiments, evaluations, attitudes, and emotions via the computational treatment of subjectivity in text. It is not our intention to review the entire body of literature concerning sentiment analysis. Indeed, such an endeavor would not be possible within the limited space available (such treatments are available in Liu (2012) and Pang & Lee (2008)). We do provide a brief overview of anonical works and techniques relevant to our study.

**What are Sentiment Lexicons?**

A sentiment lexicon  is a list of lexical features (e.g., words) which are generally labeled according to their semantic orientation as either positive or negative (Liu, 2010).

#### Using NLTK's Vader for Sentiment Analysis

**References for NLTK**: [NLTK page](http://www.nltk.org/api/nltk.sentiment.html), [NLTK Sentiment Github](https://github.com/nltk/nltk/blob/develop/nltk/sentiment/), [Vader author Github](https://github.com/cjhutto/vaderSentiment)

First, it is necessary to download `nltk` via `pip`. This can be done with the following command.

```
sudo pip install -U nltk
```

**Note**: The integration between `nltk` and `python3` was pretty painful. I could not actually install `nltk` for `python3`, so we had to default to using `python2`. The first statement below details how it was installed and then the next 3 statements show how it was used in the python file.

```{python}
import nltk

nltk.download('vader_lexicon')
from nltk.sentiment.vader import SentimentIntensityAnalyzer
```

[What is compound score?](https://github.com/cjhutto/vaderSentiment#user-content-about-the-scoring)

**[NOT CHOSEN]** [TextBlob](http://textblob.readthedocs.io/en/dev/quickstart.html#sentiment-analysis) for Python

**[NOT CHOSEN]** [Sentiment Analysis Tool](https://www.paralleldots.com) for Python

## Emotion Analysis

We tried to use ParallelDots for [Emotional Analysis Tool](https://www.paralleldots.com),  BUT there is a super strict api limit of 1000 api hits a day.

Thus, we started using the [TidyText](https://www.tidytextmining.com/) package in R.

**[TODO]** Fill the rest of this out ...

## Word Count

Based on the highest frequency of words throughout comments on the day of the election, we build a word cloud masked by the shape of the United States. [Here](https://github.com/amueller/word_cloud) is the Python package used to build the **WordCloud**.

**Note**: Similar to Vader, the integration between `WordCloud` and `python3` was pretty painful. So, we had to default to using `python2`.

## Technical Notes

### NLTK's Vader (Valence Aware Dictionary for sEntiment Reasoning)

Hutto, C.J. & Gilbert, E.E. (2014). VADER: A Parsimonious Rule-based Model for Sentiment Analysis of Social Media Text. 
Eighth International Conference on Weblogs and Social Media (ICWSM-14). Ann Arbor, MI, June 2014.

We use a combination of qualitative and quantitative methods to produce, and then empirically validate, a gold-standard  sentiment lexicon that is especially attuned to microblog-like contexts.

We find that incorporating these heuristics
improves the accuracy of the sentiment analysis engine
across several domain contexts (social media text, NY
Times editorials, movie reviews, and product reviews).
Interestingly, the VADER lexicon performs exceptionally
well in the social media domain. The correlation coefficient
shows that VADER (r  = 0.881) performs as well as
individual human raters (r  = 0.888) at matching ground
truth (aggregated group mean from 20 human raters for
sentiment intensity of each tweet).

#### METHODS

Our approach seeks to leverage the advantages of parsimonious rule-based modeling to construct a computational sentiment analysis engine that 1) works well on social media style text, yet readily generalizes to multiple domains, 2) requires no training data, but is constructed from a generalizable, valence-based, human-curated gold standard sentiment lexicon 3) is fast enough to be used online with streaming data, and 4) does not severely suffer from a speed-performance tradeoff.

##### MACHINE LEARNING

We use the Python-based machine learning algorithms from scikit-learn.org for the NB, Maximum Entropy ( makes no conditional independence assumption between features, and thereby accounts for information entropy (feature weightings), SVM-Classification (SVM-C) and SVM-Regression (SVM-R) models.

- bag of words, TF-IDF and 2 important algorithms NB and SVM

From the commit logs we can easily see that the [first version](https://github.com/nltk/nltk/blob/05c6336c3d3f34994b9597396e86bfd8d20ded4c/nltk/sentiment/sentiment_analyzer.py) sentiment_analyzer was a **Naive Bayes** classifier with **unigram features**, [one week later](https://github.com/nltk/nltk/commit/40c1f48e9f3046c989cab2edde952139c8c7e753) a **Maximum entropy** classifier was added, and [two days after](https://github.com/nltk/nltk/commit/54ab0322287a4b64395d26f05a654280b3ca9360) that facility for **bigram** features were added.

**Links**
- [Using VADER to handle sentiment analysis with social media text](http://t-redactyl.io/blog/2017/04/using-vader-to-handle-sentiment-analysis-with-social-media-text.html)

###### MACHINE LEARNING

**N-Grams**

The basic point of n-grams is that they capture the language structure from the statistical point of view, like what letter or word is likely to follow the given one. The longer the n-gram (the higher the n), the more context you have to work with. Optimum length really depends on the application – if your n-grams are too short, you may fail to capture important differences. On the other hand, if they are too long, you may fail to capture the “general knowledge” and only stick to particular cases.

They are basically a set of co-occuring words within a given window and when computing the n-grams you typically move one word forward (although you can move X words forward in more advanced scenarios). For example, for the sentence "The cow jumps over the moon". If N=2 (known as bigrams), then the bigrams would be:

* the cow
* cow jumps
* jumps over
* over the
* the moon

**Links**
[Using Bigrams to Enhance Text Classification](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.2.1938&rep=rep1&type=pdf)

**Difference between Naive Bayes and Multinomial Naive Bayes**

[Text classification using NB](https://medium.com/@theflyingmantis/text-classification-in-nlp-naive-bayes-a606bf419f8c)

[Stack Exchange Explanation](https://stats.stackexchange.com/questions/33185/difference-between-naive-bayes-multinomial-naive-bayes#answer-34002)
