---
layout: post
title: Data Science of the Apartment Search, part 1
excerpt_separator: <!--more-->
date: April 29, 2019
---

### Summary

The search for the perfect apartment is a natural fit for data science: between city data published via open-data initiatives and Craigslist listings of apartments to rent, there’s a lot of structured and semi-structured geographic data to analyze. With rent prices rising rapidly in urban areas, finding a good apartment at a reasonable price in the right neighborhood is a challenge, and being the first to visit and apply for a property confers a sizable competitive advantage. This is especially relevant in San Francisco, where legends abound about high rental prices and the crazy competition of landing a place. As the Bay Area is a place I’m considering moving to within the next couple months, using data science to identify my next apartment represents both a useful and fun challenge to take on.

<!--more-->

------

Implicit in the process of finding and renting an apartment are a number of stages where data can inform and shape actions via data-driven decision making. These include:

* Deciding how long to look/how many apartments to visit before selecting one (the secretary problem, a classic in decision science)

* Choosing geographic areas (neighborhoods, cities) in which to look for apartments (a spatial, or GIS-based data science application)

* Determining what a fair value is for a listing, and jumping on listings that are at below-market-rate (presumably via a machine-learning model such as a tree-based method, or a neural network)

	These stages themselves can be broken into individual data science tasks, which will be the focus of this series moving forward. Previous work is found here: https://towardsdatascience.com/going-dutch-how-i-used-data-science-and-machine-learning-to-find-an-apartment-in-amsterdam-part-def30d6799e4, 
https://towardsdatascience.com/using-python-to-find-myself-a-rental-home-a0b1bf6f02e, and https://www.dataquest.io/blog/apartment-finding-slackbot/, but my plan is to go deeper into the data science behind the apartment search. The idea is to incorporate the data science process from start to finish, and the structure of this will be roughly:

1) Introduction -- this post! How I’m looking at the problem.

2) Principal Component Analysis (PCA) and hierarchical clustering to understand the microclimates of the Bay Area, and compare them to a climate I’m well familiar with (Portland)

3) Creating a simulation for the secretary problem, and looking at two modifications of it: the first, how having some knowledge of the prior distribution of candidates alters the optimal hiring strategy, and the second, of how finding a candidate in the top 10% (instead of solely the best) changes the optimal strategy

4) Using the Google Maps API to determine points of interest, such as stores, to guide choices of where to live and where not to

5)Integrating public transit data, store locations, crime, and other spatial attributes to determine suitable areas in which to rent apartments (GIS-based site selection)

6) Scraping apartment listings from Craigslist, transforming this into structured data, and loading these listings into a database for future analysis

7) Analyzing these listings to understand the data and find useful insights

8) Using the listings data, creating additional features using feature engineering and fitting a machine learning model to predict that listing’s price

9) Automating notifications e.g. via Slack to notify me of apartments fitting the characteristics that I’m looking for

10) (optional) Using Slack interactive messages, AWS Gateway, and AWS Lambda to allow upvoting/downvoting apartments (for fun, I haven’t used AWS Gateway before)

11) A retrospective of this process, including a description of how this worked out!



