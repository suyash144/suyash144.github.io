---
layout: default # Or a specific 'post' layout if you create one later
title: "Naive Bayes"
date: 2025-05-23 15:00:00 +0100 # Date and time of publication
author: "Suyash Agarwal" # Optional
categories: [machine learning, research] # Optional
tags: [statistics] # Optional
---

In a recent project, I tried to implement a Naive Bayes classifier to classify neurons (recorded in a mouse brain via high-density electrophysiology probes) into match or non-match. 
The neurons are recorded across many sessions (think days) and the task is to ultimately determine how neurons recorded in one session map to neurons recorded in another. 
Declaring a pair of neurons as a 'match' means it is actually the same neuron, recorded in two different sessions. A non-match means they are different neurons. 


The information we had to do this was the following:
- the shape of the waveform the neuron produces when it spikes
- the spatial location of the neuron, as recorded by the probe

To leverage the first, we trained a neural network to look at the waveforms (these are basically sets of time-series values) and compute similarity scores for these. The neural net does this by projecting the waveforms into some latent space that captures their salient features. So the first step was training an autoencoder to do this.
We then finetuned the network using contrastive learning. I won't say much more about this as it's not the point of this post. Basically, we have a neural net that gives us waveform similarity scores, and does a good job of matching the neurons using this information. But it doesn't have the spatial information (so it looks at the shapes of the neuron's spikes but not where it is).

The way we had incorporated the spatial information before was to take the matches predicted by the neural network and then remove the ones that are spatially distant from each other. But we wanted to try something a bit smarter, that could take both waveform similarity and spatial distance into account at the same time, rather than having these two steps in series.

To this end, we thought a Naive Bayes could work with the neural network similarity and distance between two neurons as its two features. Here, each sample is a pair of neurons. I trained one of these using a kernel density estimator to get conditional distributions for each feature (conditioned on our two labels; match and non-match). All seemed well and I expected this to do at least as well as the neural network with spatial filtering and *much* better than the neural network alone. But...

<img src="{{ '/assets/images/res.png' | relative_url }}" alt="Suyash Agarwal" style="width: 50%;">
