---
layout: default # Or a specific 'post' layout if you create one later
title: "Naive Bayes: a subtle issue"
date: 2025-05-23 15:00:00 +0100 # Date and time of publication
author: "Suyash Agarwal" # Optional
categories: [machine learning, research] # Optional
tags: [statistics] # Optional
---

## Naive Bayes

TLDR: Beware when using a Naive Bayes classifier in a situation where one or more of your features has a conditional distribution that is a delta function.

### Background

In a recent project, I tried to implement a Naive Bayes classifier to classify neurons (recorded in a mouse brain via high-density electrophysiology probes) into match or non-match. 
The neurons are recorded across many sessions (think days) and the task is ultimately to determine how neurons recorded in one session map to neurons recorded in another. 
Declaring a pair of neurons as a 'match' means it is actually the same neuron, recorded in two different sessions. A non-match means they are different neurons. 


The information we had to do this was the following:
- the shape of the waveform the neuron produces when it spikes (fires an action potential)
- the spatial location of the neuron, as recorded by the probe

To leverage the first, we trained a neural network to look at the waveforms (these are basically sets of time-series values) and compute similarity scores for these. The neural net does this by projecting the waveforms into some latent space that captures their salient features. So the first step was training an autoencoder to do this.
We then finetuned the network using contrastive learning. I won't say much more about this as it's not the point of this post. Basically, we have a neural net that gives us waveform similarity scores, and does a good job of matching the neurons using this information. But it doesn't have the spatial information (so it looks at the shapes of the neurons' spikes but not where they are).

### My naive attempt at Naive Bayes

The way we had incorporated the spatial information before was to take the matches predicted by the neural network and then remove the ones that are spatially distant from each other. But we wanted to try something a bit smarter, that could take both waveform similarity and spatial distance into account at the same time, rather than having these two steps in series.

To this end, we thought a Naive Bayes could work with the neural network similarity and distance between two neurons as its two features. Here, each sample is a pair of neurons. I trained one of these using a kernel density estimator to get conditional distributions for each feature (conditioned on our two labels; match and non-match). All seemed well and I expected this to do at least as well as the neural network with spatial filtering and *much* better than the neural network alone. But...

<img src="{{ '/assets/images/res.png' | relative_url }}" alt="Suyash Agarwal" style="width: 100%;">

To explain this, the left plot is AUC (an accuracy score between 0 and 1) and the right plot is the number of matches found. The two axes are the original method (neural network similarity scores followed by spatial filtering on the x-axis) and the new Naive Bayes method (on the y-axis). Each point represents a pair of recording sessions on which our two performance metrics were computed. You can see that the Naive Bayes is not performing as well as the original method. What if we compare it to the original method without any spatial filtering? If the Naive Bayes doesn't win that comparison then something has to be wrong.

<img src="{{ '/assets/images/res2.png' | relative_url }}" alt="Suyash Agarwal" style="width: 100%;">

Hmm. The Naive Bayes (which has both waveform similarity and spatial information to work with) has a similar AUC to a method that only looks at waveform similarity. The latter also finds *more* matches, though this may just be because the spatial filtering step acts only to reduce the number of matches returned.