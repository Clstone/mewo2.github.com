---
layout: post
title: "EMI Music Hackathon: How I Did It"
description: ""
category: kaggle
tags: [hackathon]
---
{% include JB/setup %}

Last weekend, [Kaggle][kaggle] and [Data Science London][dsl] ran a second hackathon, this time focused around the [EMI One Million Interview Dataset][dataset], a large database of musical preferences. I took third place globally in this competition, and this is an attempt to explain how my model worked. The code is available [on GitHub][github].

For this competition I took a "blitz" approach. Rather than focusing on one model, trying to tweak all the performance I could out of it, I threw together a bunch of simple models, and blended their results. In the end, I combined ten separate predictions for my final submission.

Because I knew that I was going to be blending the results, for each model I retained a set of cross-validation predictions for the training set. These were used as input to the blending process, as well as to give me an idea of how well each model was performing, without having to use up my submission quota. In general, I used ten-fold cross-validation, but for models which used random forests, I simply used the out-of-bag predictions for each data point. 

### Preprocessing

As given, the data consists of a table of (user, artist, track, rating) quadruples, along with tables of data about users (demographic information, etc), and about user-artist pairs (descriptive words about each artist, whether the user owned any of their music, etc). These secondary tables were quite messy, with lots of missing data and a lack of standard values.

I generally don't enjoy data cleaning, so I did one quick pass through this data to tidy it up a little, then used it as-is for all the models. I merged the two "Good lyrics" columns, which differed only in capitalisation. For the "OWN_ARTIST_MUSIC" column, I collapsed the multiple encodings for "Don't know". Similarly, I collapsed several of the levels of the "HEARD_OF" column. The responses for "LIST_OWN" and "LIST_BACK" needed to be converted to numbers, rather than the mish-mash of numeric and text values which were there to begin with.

To fill in missing values, I used the median value for numeric columns, and the most common value for categorical columns. I then joined these tables with the training data.

In most cases, the results were aided by first removing "global effects". I subtracted the overall mean rating, and then estimated effects for users and tracks, with Bayesian priors which reduced these effects towards zero for poorly sampled users and tracks. These effects were then added back in after the model prediction had been made.

### Chuck it all in a Random Forest/GBM/Linear Regression

The first thing I tried was an attempt to mimic Ben Hamner's success in the last hackathon, by throwing everything into a random forest and hoping for the best. It turned out that while this was a pretty good approach, it was also extremely slow. I was only able to run a limited number of trees, with a reduced sample size, so the results probably weren't as good as they could have been. I also originally ran this without removing global effects, and didn't have time to go back and do it again.

As variations on the same theme, I tried a GBM and a simple linear regression. These were less successful, but much faster.

### Partitioning the data

While a simple linear regression seemed unlikely to be successful, I thought it might be reasonable to try a separate regression for each artist. The results were surprisingly good.

Given that the random forest was so successful, and that the per-artist linear regression was quite good, it seemed like a good idea to try a per-artist random forest approach. This was also good, but not as good as I'd hoped.

Weirdly, the per-artist approach was much more successful than a per-track approach. A linear model on a per-track basis was much worse than the per-artist model – so bad that I didn't even try a random forest.

### SVD

The models I've used up to now have been the kind of tools that one would use on a generic machine learning problem. However, the kind of user-item-rating data we're given here leads to what is called a "collaborative filtering" problem, for which there are a number of specialised techniques available. The most successful approaches in the past have come from matrix factorisation models, the simplest of which is SVD (singular value decomposition).

The particular form of SVD I used is one which was developed for the Netflix Prize competition by Simon Funk, who wrote an excellent [blog post][funk] about how it works, and its implementation. Oddly, there doesn't seem to be a standard implementation available in R, so I wrote my own in C and interfaced with that.

I ended up using results from two separate SVD runs: a quick early one, and a longer one with more features. The results were not fantastic, but they contributed greatly to the blend.

### Nearest neighbour

Another common approach to collaborative filtering problems is nearest-neighbour filtering. I calculated a distance measure between tracks, based on the correlation between ratings given by people who rated both tracks. To predict new ratings, I looked through the user's rating history and calculated a weighted average of the "most similar" tracks this user had rated. The results were fairly disappointing — most users didn't have enough ratings to make this approach viable.

### Demographics

As a final attempt at adding some variety to the blend, I tried an approach based purely on the demographic info given. I divided the users into five age quantiles, then for each track calculated the average score for each age quantile and gender. These turned out to be pretty terrible predictors of the actual ratings.

### Blending

For the final blend I used a neural network with five hidden units, and skip-layer connections. Given that there were only ten inputs and over a hundred thousand training examples, overfitting was never a huge concern. However, a small weight decay did help with convergence.

### Code, etc

The code is now available [on GitHub][github]. This is a slightly cleaned-up version of the actual code I wrote during the competition – most of the results I actually generated were done with a lot of tweaking hard-coded parameters and so on. However, the overall results should be pretty similar to what I submitted. As always, questions and comments are welcome, but suggestions for improvement are probably not going to be followed up.

[github]: http://www.github.com/mewo2/musichackathon/
[kaggle]: http://www.kaggle.com/
[dsl]: http://datasciencelondon.org/
[dataset]: http://musicdatascience.com/emi-million-interview-dataset/
[funk]: http://sifter.org/~simon/journal/20061211.html
