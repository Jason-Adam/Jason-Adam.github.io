---
layout: post  
mathjax: true  
comments: true  
title: NLP Text Tokenization  
tags: [NLP]  
---
This is a brief tutorial on manual text tokenization of a document corpus using the `nltk` library.  

The corpus used in this tutorial was created by a crawler I wrote utilizing the `scrapy` library. The repo for that project can be found [here](https://github.com/Jason-Adam/stack-crawler). The purpose of the crawler was to build a corpus of top StackOverflow questions related to **Artificial Intelligence**, **Machine Learning**, and **NLP** that could be used for further modeling work.  

## Imports


```python
import os
import re
import string

from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords

import pandas as pd
import pymongo

%load_ext blackcellmagic
```

    The blackcellmagic extension is already loaded. To reload it, use:
      %reload_ext blackcellmagic



```python
# Global Variables
COLLECTIONS = ["ai_questions", "ml_questions", "nlp_questions"]
```

The crawler is setup to write to a local MongoDB on my computer. The following section retrieves each document from the corpus and loads it to pandas dataframe.  

## Retrieve Corpus from MongoDB


```python
# Retrieve questions from MongoDB
def get_question_txt(collection: str) -> list:
    client = pymongo.MongoClient(host="localhost", port=27017)
    db = client["stackoverflow"]
    ai = pymongo.collection.Collection(database=db, name=collection)
    cursor = ai.find({})
    return [(collection, i["url"], i["title"], i["question"]) for i in cursor]


# Write corpus to pandas df
def convert_corpus(collection: list) -> dict:
    corp = {
        "so_tag": [],
        "title": [],
        "url": [],
        "question": [],
    }

    for c in collection:
        docs = get_question_txt(c)
        for q in docs:
            corp["so_tag"].append(q[0])
            corp["url"].append(q[1])
            corp["title"].append(q[2])
            corp["question"].append(q[3])

    return pd.DataFrame(corp)
```

The code above creates a connection to the local MongoDB, loops through the document collections and loads all documents with their title and URL as a list of tuples. The second function converts the list of tuples to a pandas dataframe.


```python
cdf = convert_corpus(COLLECTIONS)
print(cdf.shape)
```

    (1494, 4)


We can see that the documents loaded correctly and that there are 1,494 of them in the entire corpus. On StackOverflow, questions can have more than one tag, so we'll remove any duplicate documents so that we have independent samples for each class.


```python
cdf.drop_duplicates(subset="url", keep=False, inplace=True)
cdf.reset_index(inplace=True, drop=True)
print(cdf.shape)
```

    (1313, 6)



```python
cdf.head()
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
      <th>so_tag</th>
      <th>title</th>
      <th>url</th>
      <th>question</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ai_questions</td>
      <td>What is the optimal algorithm for the game 2048?</td>
      <td>/questions/22342854/what-is-the-optimal-algori...</td>
      <td>I have recently stumbled upon the game 2048. Y...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ai_questions</td>
      <td>Pacman: how do the eyes find their way back to...</td>
      <td>/questions/3148435/pacman-how-do-the-eyes-find...</td>
      <td>I found a lot of references to the AI of the g...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ai_questions</td>
      <td>What is the best Battleship AI?</td>
      <td>/questions/1631414/what-is-the-best-battleship-ai</td>
      <td>Battleship! Back in 2003 (when I was 17), I co...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ai_questions</td>
      <td>source of historical stock data [closed]</td>
      <td>/questions/754593/source-of-historical-stock-data</td>
      <td>I'm trying to make a stock market simulator (p...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ai_questions</td>
      <td>What are good examples of genetic algorithms/g...</td>
      <td>/questions/1538235/what-are-good-examples-of-g...</td>
      <td>Genetic algorithms (GA) and genetic programmin...</td>
    </tr>
  </tbody>
</table>
</div>



We can see that we've lost some records but not many.  

## Tokenization  
Tokenization is the first step in any NLP pipeline. In general terms, tokenization is the task of segmenting documents. The most common way to tokenize documents is to break them into individual words. Once we break down documents to discrete elements (tokens), we can use them for subsequent modeling and analysis. Some of the main challenges involved with tokenization are:  
* Stop words (i.e. the, and, etc.)  
* Punctuation  
* Removal of Numbers  
* Removal of tags (i.e. HTML tags if document is scraped)  

For this particular corpus, I've stripped off all HTML tags during the scraping process. We will focus on the other components in the next step.  


```python
# Regex for removing punctuation
re_punc = re.compile("[%s]" % re.escape(string.punctuation))

# Tokenize text and join back as one string
def tokenize_txt(txt: str):
    tok = word_tokenize(txt)
    tok = [w.lower() for w in tok]
    tok = [re_punc.sub("", w) for w in tok]
    tok = [word for word in tok if word.isalpha()]
    stop_words = set(stopwords.words("english"))
    tok = [w for w in tok if w not in stop_words]
    return " ".join(tok)
```

The above function takes in a string (i.e. one of the documents) and first splits it into a list of individual words. The list of words in converted to lowercase, all punctuation is removed, and any numeric values are removed. Stop words are removed before combining the words back together as a cleaned document that can be used in future modeling work via `scikit-learn`, `keras`, `gensim`, etc.  

We could have just returned a list of words, but that on it's own isn't as useful as the fully cleaned document.


```python
# Tokenize questions
cdf["cleaned_question"] = cdf["question"].apply(tokenize_txt)

# Create document id based on unique numeric value from URL field
cdf["doc_id"] = cdf.apply(lambda r: r["url"].split("/")[2], axis=1)
```

Below we can see a single document before and after.  

**Before**


```python
print(cdf.iloc[0, 3])
```

    I have recently stumbled upon the game 2048. You merge similar tiles by moving them in any of the four directions to make "bigger" tiles. After each move, a new tile appears at random empty position with a value of either 2 or 4. The game terminates when all the boxes are filled and there are no moves that can merge tiles, or you create a tile with a value of 2048. One, I need to follow a well-defined strategy to reach the goal. So, I thought of writing a program for it. My current algorithm: What I am doing is at any point, I will try to merge the tiles with values 2 and 4, that is, I try to have 2 and 4 tiles, as minimum as possible. If I try it this way, all other tiles were automatically getting merged and the strategy seems good. But, when I actually use this algorithm, I only get around 4000 points before the game terminates. Maximum points AFAIK is slightly more than 20,000 points which is way larger than my current score. Is there a better algorithm than the above?


**After**


```python
print(cdf.iloc[0, 4])
```

    recently stumbled upon game merge similar tiles moving four directions make bigger tiles move new tile appears random empty position value either game terminates boxes filled moves merge tiles create tile value one need follow welldefined strategy reach goal thought writing program current algorithm point try merge tiles values try tiles minimum possible try way tiles automatically getting merged strategy seems good actually use algorithm get around points game terminates maximum points afaik slightly points way larger current score better algorithm


## Summary  
Tokenization is a critical step in any NLP pipeline, and great attention should be paid to it. This is a brief summary of some of the challenges encountered when tokenizing documents, but hopefully it gives some guidance on starting the process.
