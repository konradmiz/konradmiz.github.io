---
layout: post
title: Predicting Wine Varieties, Part 1 -- Naive Bayes
excerpt_separator: <!--more-->
date: September 07, 2018
---

## Summary

This is the first post in a series where I use different algorithms to try to classify, i.e. predict a wine variety given its description. I start off with a simple yet powerful algorithm called *Naive Bayes*. This algorithm is particularly easy to build (and use) from scratch and so though the code is not intended for use in production, it was fun to write. It performed admirably (33.16% accuracy on a validation set of 4917 rows representing 40 different varieties, given a description of more than 140 characters). It will be compared head-to-head with the other algorithms on the test set at the conclusion of this series!

<!--more-->
### Introduction

I first played with the wine reviews dataset a couple weeks ago (post here: <https://konradmiz.github.io/Identifying-wine-qualities/>), by finding words that characterize popular varieties of wine, and making a bunch of different word clouds. Here, I try to do the more challenging task: predicting the variety of wine given a set of words in the description. This problem is known as text classification, and has a variety of usages, ranging from spam filtering to determining disputed authorship of several Federalist papers (link here: <https://link.springer.com/chapter/10.1007/978-1-4612-5098-2_70>).

There are many different approaches to text classification, and I'll be exploring some of them in this and subsequent posts. For me, the first algorithm that comes to mind for text classification is Naive Bayes, so I start with that here. Naive Bayes is a simple yet powerful algorithm that uses word frequency in each training class to determine the likelihood of the test observation belonging to each class.

It relies on Bayes' rule that:

*P*(*A*\|*B*) = $\\frac{P(B\|A)P(A)}{P(B)}$

Since P(B) is constant it can be ignored. By extending to multiple dimensions (and using the chain rule of probability to do so), it can be shown that the probability of class i is

*P*(*C*<sub>*i*</sub>\|*x*<sub>1</sub>, *x*<sub>2</sub>) = *P*(*x*<sub>1</sub>\|*C*<sub>*i*</sub>)*P*(*x*<sub>2</sub>\|*C*<sub>*i*</sub>)*P*(*C*<sub>*i*</sub>), or extending to n dimensions,

*P*(*C*<sub>*i*</sub>\|*x*<sub>1</sub>, *x*<sub>2</sub>, ..., *x*<sub>*n*</sub>) = *P*(*C*<sub>*i*</sub>) $\\prod<sub>i=1</sub><sup>n</sup>$ *P*(*x*<sub>*j*</sub>\|*C*<sub>*i*</sub>).


In this case, I am trying to find the class of the wine, *P*(*C*<sub>*i*</sub>), given the words in the description *x*<sub>1</sub>, ..., *x*<sub>*n*</sub>. 

To do so I can find the overall frequency of the varieties within the dataset, *P*(*C*<sub>*i*</sub>), and take the product of the likelihood of each word in the description belonging to a each wine variety, where each 

*P*(*x*<sub>*j*</sub>\|*C*<sub>*i*</sub>) is the observed frequency of word *j* belonging to class *i*.


The word *Naive* comes from the model assumption that a word's occurrence in a piece of text is independent of every other word in the text. While this typically does not hold in any practical setting (words are correlated with each other: if a text contains the word 'wine' it is more likely to also contain the word 'drink'), the model still performs quite well in many applications.

Data Import
-----------

``` r
library(readr)
library(dplyr)
library(stringr)
library(ggplot2)
library(tidytext)
library(tidyr)

set.seed(1)

wine <- read_csv("docs/Wine 130k reviews.csv") %>%
  select(-X1) %>% # this is a row number column and can be dropped
  distinct()
```

Data Processing
---------------

Accurately predicting something is a challenging task for which a lot of data is typically required. Fortunately, the wine reviews are generally verbose, with mean 242.811 +/- 67.142 characters. I decided to select only verbose reviews, i.e., those that wouldn't fit on a tweet -- more than 140 characters. There is a possible tradeoff in the choice of description length: longer descriptions are more informative, but there are fewer of them to train the model on; shorter descriptions are more frequent, but predicting the class of a shorter description is more challenging.

``` r
long_descriptions <- wine %>%
  filter(nchar(description) > 140)
```

After filtering out those descriptions, there were still 705 wine varieties, so I wanted to limit that number as well. To do so, I filtered varieties that were reviewed 500 times or more.

``` r
popular_long_descriptions <- long_descriptions %>%
  group_by(variety) %>%
  add_count() %>%
  filter(n >= 500) %>%
  select(variety, description)
```

Filtering out wines that had been reviewed 500 times or more still left 40 different wine varieties, and left me 98,343 reviews.

I then split the data into training, validation, and testing sets. I chose a breakdown of 70% for the training set, 5% validation, and 25% testing.

``` r
# 70% of each wine's reviews go to the training data
train_data <- popular_long_descriptions %>%  
  group_by(variety) %>%
  sample_frac(size = 0.70, replace = FALSE) %>%
  ungroup()


# the rest go to the test/validation
test_validation <- popular_long_descriptions %>% 
  anti_join(train_data)

# 1/6 of the training data is 5% of the original
validation_data <- test_validation %>%
  group_by(variety) %>%
  sample_frac(size = 1/6) %>%
  ungroup() %>%
  sample_n(size = nrow(.)) %>%
  mutate(Row = row_number())

# the remainder is used for testing
test_data <- test_validation %>% 
  anti_join(validation_data) %>%
  ungroup() %>%
  sample_n(size = nrow(.)) %>%
  mutate(Row = row_number())
```

For the Naive Bayes algorithm, I need several different pieces of data:

1.  given a wine variety, its overall likelihood of being within the dataset, i.e. *P*(*C*<sub>*i*</sub>)

2.  given a certain word, its overall likelihood of belonging to a certain wine variety, i.e. *P*(*x*<sub>*j*</sub>|*C*<sub>*i*</sub>)

Calculating (1), the priors, is a straightforward dplyr task, as it's just the frequency of the wine varieties:

``` r
priors <- train_data %>% 
  group_by(variety) %>% 
  count() %>% 
  ungroup() %>% 
  mutate(Frac = round(n/sum(n), 4))
```

From this, we can see the most frequent wine (Pinot Noir, 0.1203) and the least frequent wine (Chenin Blanc, 0.0053) in the data. Our *a priori* assumptions are that these prior frequencies are the likelihoods that a wine in the validation or training set belong to that variety.

To calculate (2), the likelihoods, we need to wrangle the description data, splitting from grouping sets of words per review to by variety. To do this, like with the wine wordclouds post, we need to unnest the tokens.

``` r
wine_types <- c(unique(tolower(wine$variety)), 
                "barolo", "barbaresco", "cab", "cabs", "sauv", 
                "chablis",  "grÃ¼ner", "gewÃ¼rz",  
                "pinot", "pinots", "noir", "noirs",
                "sb", "sbs", "blanc", "syrahs",
                "zin", "zins", "zin's")

train_tokens <- train_data %>%
  unnest_tokens(word, description) %>%
  anti_join(stop_words) %>% # remove stop words
  #remove words in the description that correspond to any wine variety
  filter(!str_detect(regex(word, ignore_case = TRUE), 
                     regex(variety, ignore_case = TRUE)),
         ! word %in% wine_types) %>%
  count(variety, word) %>%
  ungroup()
```

Once that is done, we have the counts of the words for each variety. We can group by word and calculate the frequency with which a word is used to describe a specific wine variety.

``` r
likelihoods <- train_tokens %>%   
  group_by(word) %>%
  mutate(N = sum(n),
         Frac = n/N) %>%
  ungroup(word)
```

We can see what a random row of the dataframe looks like:

``` r
sample_n(likelihoods, 1)
```

    ## # A tibble: 1 x 5
    ##   variety      word        n     N   Frac
    ##   <chr>        <chr>   <int> <int>  <dbl>
    ## 1 Chenin Blanc extreme     3   164 0.0183

The lowercase n represents the number of times that word appeared for that variety of wine, the uppercase N represents the total occurences within the datase, and the Frac represents the probability that the wine is of that variety, given that word in the description.

One consideration in a text mining task like this is that there may be words in the validation or testing datasets that do not appear in the training data for that variety of wine. Instead of treating these as impossible events with a probability zero (since they did actually occur), instead we need to define a non-zero probability for that word being used to describe that wine.

This process is called Laplace or additive smoothing, and involves using a uniform distribution, such that we assign every not-seen word a probability of occurence of $\frac{1}{1 + \# of varieties + \# of distinct words}$.

``` r
num_varieties <- nrow(priors)

num_words <- train_tokens %>%
  distinct(word) %>%
  nrow()

new_prob <- 1/(num_varieties + num_words + 1)
```

Now that we have the priors (*P*(*C*<sub>*i*</sub>)) and the likelihoods (*P*(*x*<sub>*j*</sub>|*C*<sub>*i*</sub>)), the classification can now be done. For each description in the validation set, I perform a similar data processing task as I did with the training set:

-   tokenize the descriptions into individual words

-   remove stop words and a handful of hand-selected wine variety words

With the words I'll be using to predict the variety now prepared, I filter the likelihoods to include only words in the description.

For each variety, I pull the words that had been used for the description and calculate the product of their likelihoods (i.e., applying the chain rule to the likelihoods). Since not every word was used for every variety, I need to fill in the number of words that weren't used. Not doing so would lead to an overestimation of the likelihood and produce the wrong classification.

For an example of this, imagine trying to classify a description into one of three classes A,B,C. The description is 100 words long, and the probabilities are like this:

A: Only one word matches from the description: *P*(*x*<sub>1</sub>|*A*) = 0.5 and *P*(*x*<sub>2</sub>, ..., *x*<sub>100</sub>|*A*) = 0

B: All words match the description, and *P*(*x*<sub>1</sub>|*B*) = 0.4, and *P*(*x*<sub>2</sub>, ..., *x*<sub>100</sub>|*B*) = 0.5

C: All words match the description, and *P*(*X*<sub>1</sub>|*C*) = 0.1, and *P*(*x*<sub>2</sub>, ..., *x*<sub>100</sub>|*C*) = 0.5

By ignoring the words that didn't match, the likelihood of the word belonging to A would be `0.5`, while the likelihood for belonging to B would be 0.4 \* (0.5)<sup>99</sup> = 6.310887210^{-31} and the likelihood of belonging to C would be 0.1 \* (0.5)<sup>99</sup> = 1.577721810^{-31}. So while clearly class B should be the correct one, since we didn't include 99 (missing) term probabilities in A, A has the incorrectly highest likelihood.

To deal with this issue, we take the product of the probability from the Laplace smoothing *n* times, where *n* is the number of words in the description - the number of words that matched for that variety.

Once the product of the likelihoods has been taken, we multiply the result by the prior likelihood of the wine belonging to that variety. To render the classification verdict, we take the highest probability for the wine variety given the description and give it that label.

``` r
# the results of the validation will be stored here
validation_df <- data_frame(Predicted = NA,
                            Likelihood = NA,
                            Actual = validation_data$variety)

# This loop goes row-by-row through the entire dataset and 
# classifies each oservation. It filters the records based on row number 
# assigned in a previous chunk of code. It unnests the description, 
# removes stop words, and removes wine names. 

# After that, it filters word matches from the likelihoods dataframe. 
# Then the likelihood of each variety given the descriptive words is calculated. 
# The words from the likelihoods are further filtered while looping through each variety. 
# The variety with the highest likelihood is chosen and recorded in the validation dataframe.  

# The prod function is what implements the probabilty (multiplication) chain rule

for (j in 1:nrow(validation_data)) {
  
  one_obs_description <- validation_data %>%
    filter(Row == j) %>%
    unnest_tokens(word, description) %>%
    anti_join(stop_words, by = "word") %>%
    filter(!str_detect(regex(word, ignore_case = TRUE), 
                       regex(variety, ignore_case = TRUE)),
          !(word %in% wine_types))
  
  given_words <- likelihoods %>%
    filter(word %in% one_obs_description$word)
  
  variety_vector <- vector(mode = "numeric", length = num_varieties)
  names(variety_vector) <- priors$variety
  
  for (i in 1:num_varieties) {
    # Filters words in the likelihoods df that matched variety i
    # and returns a vector of likelihoods (p(variety i | word))
    
    found_words <- given_words %>% 
      filter(variety == names(variety_vector[i])) %>%
      pull(Frac)
    
    found_prob <- prod(found_words) # likelihood given observed words
    
    #number of words in description not found in the training data
    not_found_words <- length(one_obs_description$word) - length(found_words) 
    
    # likelihood given unobserved words
    not_found_prob <- prod(rep(x = new_prob, times = not_found_words))
    
    # overal likelihood
    variety_vector[i] <- found_prob * not_found_prob
  }
  
  # multiplying by the prior probability
  variety_vector_priors <- variety_vector * priors$Frac 
  
  # selecting the most likely variety 
  validation_df$Predicted[j] <- names(variety_vector_priors[which(variety_vector_priors == max(variety_vector_priors))])
  validation_df$Likelihood[j] <- variety_vector_priors[which(variety_vector_priors == max(variety_vector_priors))]
  
}

validation_df <- validation_df %>%
  mutate(Correct = ifelse(Predicted == Actual, TRUE, FALSE))
```

On the validation set, the Naive Bayes algorithm had an accuracy of 33.157%. Neat!

With the validation now done, it's possible to inspect the model results and analze more closely how it performed.

``` r
# True prevalance of varieties within the validation set
validation_types <- validation_data %>% 
  group_by(variety) %>%
  count() %>%
  ungroup() %>%
  mutate(Actual = round(n/sum(n), 3)) %>%
  select(-n)

# Accurately predicted prevalances within the validation set
accurate_predictions <- validation_df %>%
  filter(Correct == TRUE) %>%
  group_by(Actual) %>%
  count() %>%
  ungroup() %>%
  mutate(Correct = round(n/sum(n), 3)) %>%
  select(-n)

# All predictions prevalence within the validation set
all_predictions <- validation_df %>%
  group_by(Predicted) %>%
  count() %>%
  ungroup() %>%
  mutate(Prediction = round(n/sum(n), 3)) %>%
  select(-n)

# These tables joined together
results <- validation_types %>%
  left_join(accurate_predictions, by = c("variety" = "Actual")) %>%
  replace_na(list(Correct = 0)) %>%
  left_join(all_predictions, by = c("variety" = "Predicted")) %>%
  replace_na(list(Prediction = 0))
```

Somewhat surprisingly, though there were 40 distinct wine varieties, only 17 were predicted by the model, of which 16 had at least one accurate prediction. Indeed, the top 4 predicted varieties were responsible for 94% of the total predictions despite only making up 39% of the data.

Nonetheless, 33.157% is a substantial improvement over simply always predicting the most common variety of wine (which would give an accuracy of 12%).
These are interesting results which suggest that the Naive Bayes model does well at picking apart the differences between Cabernet Sauvignon, Chardonnay, Pinot Noir, and Red Blend wines, but does not do well at predicting other varieties, especially the less common ones. This model performance will be analyzed further in the last post of the series when I'll be comparing the algoithms head-to-head.
