[![](https://img.shields.io/pypi/v/top2vec.svg)](https://pypi.org/project/top2vec/)
[![](https://img.shields.io/pypi/l/top2vec.svg)](https://github.com/ddangelov/Top2Vec/blob/master/LICENSE)
[![](https://readthedocs.org/projects/top2vec/badge/?version=latest&token=0c691c6cc79b4906e35e8b7ede01e815baa05041d048945fa18e26810a3517d7)](https://top2vec.readthedocs.io/en/latest/?badge=latest)

Top2Vec
=======

Topic2Vector is an algorithm for topic modeling. It automatically detects topics present in text
and generates jointly embedded topic, document and word vectors. Once you train the Top2Vec model 
you can:
* Get number of detected topics.
* Get topics.
* Search topics by keywords.
* Search documents by topic.
* Find similar words.
* Find similar documents.

Benefits
--------
1. Automatically finds number of topics.
2. No stop words required.
3. No need for stemming/lemmatizing.
4. Works on short text.
5. Creates jointly embedded topic, document, and word vectors. 
6. Has search functions built in.

How does it work?
-----------------

The assumption the algorithm makes is that many semantically similar documents
are indicative of an underlying topic. The first step is to create a joint embedding of 
document and word vectors. Once documents and words are embedded in a vector 
space the goal of the algorithm is to find dense clusters of documents, then identify which 
words attracted those documents together. Each dense area is a topic and the words that
attracted the documents to the dense area are the topic words.

### The Algorithm:

**1. Create jointly embedded document and word vectors using [Doc2Vec](https://radimrehurek.com/gensim/models/doc2vec.html).**
>Documents will be placed close to other similar documents and close to the most distinguishing words.

![](https://raw.githubusercontent.com/ddangelov/Top2Vec/master/images/doc_word_embedding.svg?sanitize=true)

**2. Create lower dimensional embedding of document vectors using [UMAP](https://github.com/lmcinnes/umap).**
>Document vectors in high dimensional space are very sparse, dimension reduction helps for finding dense areas. Each point is a document vector.

![](https://raw.githubusercontent.com/ddangelov/Top2Vec/master/images/umap_docs.png)

**3. Find dense areas of documents using [HDBSCAN](https://github.com/scikit-learn-contrib/hdbscan).**
>The colored areas are the dense areas of documents. Red points are outliers that do not belong to a specific cluster.

![](https://raw.githubusercontent.com/ddangelov/Top2Vec/master/images/hdbscan_docs.png)

**4. For each dense area calculate the centroid of document vectors in original dimension, this is the topic vector.**
>The red points are outlier documents and do not get used for calculating the topic vector. The purple points are the document vectors that belong to a dense area, from which the topic vector is calculated. 

![](https://raw.githubusercontent.com/ddangelov/Top2Vec/master/images/topic_vector.svg?sanitize=true)

**5. Find n-closest word vectors to the resulting topic vector**
>The closest word vectors in order of proximity become the topic words. 

![](https://raw.githubusercontent.com/ddangelov/Top2Vec/master/images/topic_words.svg?sanitize=true)

Installation
------------

The easy way to install Top2Vec is:

    pip install top2vec


Usage
-----

```python

from top2vec import Top2Vec

model = Top2Vec(documents)
```
Parameters:

  * ``documents``: Input corpus, should be a list of strings.
  
  * ``speed``: This parameter will determine how fast the model takes to train. 
    The 'fast-learn' option is the fastest and will generate the lowest quality
    vectors. The 'learn' option will learn better quality vectors but take a longer
    time to train. The 'deep-learn' option will learn the best quality vectors but 
    will take significant time to train.  
    
  * ``workers``: The amount of worker threads to be used in training the model. Larger
    amount will lead to faster training.
    
Example
-------

### Train Model
Train a Top2Vec model on the 20newsgroups dataset.
```python

from top2vec import Top2Vec
from sklearn.datasets import fetch_20newsgroups

newsgroups = fetch_20newsgroups(subset='all', remove=('headers', 'footers', 'quotes'))

model = Top2Vec(documents=newsgroups.data, speed="learn", workers=8)

```
### Get Number of Topics
This will return the number of topics that Top2Vec has found in the data.
```python

model.get_num_topics()
>>> 77

```

### Get Topics 
This will return the topics.
```python
topic_words, word_scores, topic_nums = model.get_topics(77)

```
Returns:

  * ``topic_words``: For each topic the top 50 words are returned, in order
    of semantic similarity to topic.
  
  * ``word_scores``: For each topic the cosine similarity scores of the
    top 50 words to the topic are returned.  
    
  * ``topic_nums``: The unique index of every topic will be returned.
  
### Search Topics
We are going to search for topics most similar to **medicine**. 
```python

topic_words, word_scores, topic_scores, topic_nums = top2vec.search_topics(keywords=["medicine"], num_topics=5)
```
Returns:
  * ``topic_words``: For each topic the top 50 words are returned, in order
    of semantic similarity to topic.
  
  * ``word_scores``: For each topic the cosine similarity scores of the
    top 50 words to the topic are returned.  
    
  * ``topic_scores``: For each topic the cosine similarity to the search keywords will be returned.
  
  * ``topic_nums``: The unique index of every topic will be returned.

```python

topic_nums
>>> [21, 29, 9, 61, 48]

topic_scores
>>> [0.4468, 0.381, 0.2779, 0.2566, 0.2515]
```
> Topic 21 was the most similar topic to "medicine" with a cosine similarity of 0.4468. (Values can be from least similar 0, to most similar 1)

### Generate Word Clouds

Using a topic number you can generate a word cloud. We are going to genenarate word clouds for the top 5 most similar topics to our **medicine** topic search from above.  
```python
# generate word cloud for topic_nums from "medicine" keyword search on topics
for topic in topic_nums:
    model.generate_topic_wordcloud(topic)
```
![](https://raw.githubusercontent.com/ddangelov/Top2Vec/master/images/topic21.png)
![](https://raw.githubusercontent.com/ddangelov/Top2Vec/master/images/topic29.png)
![](https://raw.githubusercontent.com/ddangelov/Top2Vec/master/images/topic9.png)
![](https://raw.githubusercontent.com/ddangelov/Top2Vec/master/images/topic61.png)
![](https://raw.githubusercontent.com/ddangelov/Top2Vec/master/images/topic48.png)

### Search Documents by Topic

We are going to search by topic 48, a topic that appears to be about science.
```python
documents, document_scores, document_nums = top2vec.search_documents_by_topic(topic_num=48, num_docs=5)
```
Returns:
  * ``documents``: The documents in a list, the most similar are first.  
    
  * ``doc_scores``: Semantic similarity of document to topic. The cosine similarity of the
    document and topic vector.
  
  * ``doc_nums``: Indexes of documents in the input corpus of documents.
  
For each of the returned documents we are going to print its content, score and document number.
```python
documents, document_scores, document_nums = top2vec.search_documents_by_topic(topic_num=48, num_docs=5)
for index in range(0,len(document_nums)):
    print(f"Document: {document_nums[index]}, Score: {document_scores[index]}")
    print("-----------")
    print(documents[index])
    print("-----------")
```


    Document: 15227, Score: 0.6322
    -----------
      Evolution is both fact and theory.  The THEORY of evolution represents the
    scientific attempt to explain the FACT of evolution.  The theory of evolution
    does not provide facts; it explains facts.  It can be safely assumed that ALL
    scientific theories neither provide nor become facts but rather EXPLAIN facts.
    I recommend that you do some appropriate reading in general science.  A good
    starting point with regard to evolution for the layman would be "Evolution as
    Fact and Theory" in "Hen's Teeth and Horse's Toes" [pp 253-262] by Stephen Jay
    Gould.  There is a great deal of other useful information in this publication.
    -----------

    Document: 14515, Score: 0.6186
    -----------
    Just what are these "scientific facts"?  I have never heard of such a thing.
    Science never proves or disproves any theory - history does.

    -Tim
    -----------
    
    Document: 9433, Score: 0.5997
    -----------
    The same way that any theory is proven false.  You examine the predicitions
    that the theory makes, and try to observe them.  If you don't, or if you
    observe things that the theory predicts wouldn't happen, then you have some 
    evidence against the theory.  If the theory can't be modified to 
    incorporate the new observations, then you say that it is false.

    For example, people used to believe that the earth had been created
    10,000 years ago.  But, as evidence showed that predictions from this 
    theory were not true, it was abandoned.
    -----------
    
    Document: 11917, Score: 0.5845
    -----------
    The point about its being real or not is that one does not waste time with
    what reality might be when one wants predictions. The questions if the
    atoms are there or if something else is there making measurements indicate
    atoms is not necessary in such a system.

    And one does not have to write a new theory of existence everytime new
    models are used in Physics.
    -----------
    
    ...

