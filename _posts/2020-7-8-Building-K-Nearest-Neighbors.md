---
layout: post
title: Building K Nearest Neighbors
tags: [quake, Machine_Learning]
comments: true
---


# K Nearest Neighbors and Quake

The K Nearest Neighbors algorithm is a relatively simple unsupervised machine learning
algorithm. It takes in training data, and then when given a new entry returns the closest
entry from the training data. For more information check out the [wikipedia article](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm).


The first thing I tried was to manually create it from scratch, with little regard for run time.
When working with this implementation I decided to only use one-dimensional data. I was able to
put together a nested for loop. It was able to find the closest number to the one provided but was
limited to only one dimension. To jump up to two requires using the [Euclidian distance](https://en.wikipedia.org/wiki/Euclidean_distance)
between points.


To find that I decided to use a slightly less naive implementation. I used the `distance_matrix`
method in the spatial module of the scipy library. This method returns the Euclidian distance between
all the points described by the two arrays provided. Scipy is implemented such that these calculations are
done much more efficiently than even the simple 1d algorithm that I had done and was faster even across
higher-dimensional data.


This new method was so effective that I decided to pull out the project I recently called finished.
[Quake Labs](quake-ds-app.heroku.com) is an earthquake monitoring app that gets its data from two sources.
There is some overlap in the data between the sources, but there are slight differences in reporting that
make it difficult to line up quakes between the two. I had initially used the `sklearn` implementation of
k_nearest_neighbors, but for some reason, it wasn't able to find matches in the data. I decided to try my
version on the data to see if I could get any better results. I used the API that I had built to pull down the last
month of data, it came to about 40 earthquakes from each source. I also decided to be a bit pickier about my data
and instead of matching across everything I only looked at the location and magnitude of quakes. I was surprised to find that the algorithm found about 30 matches. Considering that not all of the earthquakes were lined up this number makes a lot of sense.


I then connected directly with my database and pulled down all of the earthquakes observed to line everything up. In one source I had about 40,000 observations and in the other about 100,000. However, after running for a few seconds my algorithm came back with everything it had found, 51 observations. This was not the right number, so its back to the drawing board with this one. I'll post more once I have been able to spend some time with it.
