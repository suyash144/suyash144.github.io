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

<img src="{{ '/assets/images/posts/res.png' | relative_url }}" alt="Suyash Agarwal" style="width: 100%;">

To explain this, the left plot is AUC (an accuracy score between 0 and 1) and the right plot is the number of matches found. The two axes are the original method (neural network similarity scores followed by spatial filtering on the x-axis) and the new Naive Bayes method (on the y-axis). Each point represents a pair of recording sessions on which our two performance metrics were computed. You can see that the Naive Bayes is not performing as well as the original method. What if we compare it to the original method without any spatial filtering? If the Naive Bayes doesn't win that comparison then something has to be wrong.

<img src="{{ '/assets/images/posts/res2.png' | relative_url }}" alt="Suyash Agarwal" style="width: 100%;">

Hmm. The Naive Bayes (which has both waveform similarity and spatial information to work with) has a similar AUC to a method that only looks at waveform similarity. The latter also finds *more* matches, though this may just be because the spatial filtering step acts only to reduce the number of matches returned.


### Training without labels: an aside

Before we can understand what is going wrong here, I need to detail how exactly I trained the Naive Bayes. This requires labelled data, which is something that we always struggle with in this project. We never actually have ground truth neuron labels, so the machine learning methods we could use were either self-supervised (training the autoencoder) or they used a quasi-ground truth that we could construct from the fact that within a session, we know which spikes come from which neuron thanks to upstream spike-sorting software. We also have two copies of each neuron's waveform (averaged over the first and second half of the session). This means we can do contrastive learning if we train only on within-session data, because we have pairs of neurons which are the same and pair of neurons which are different (at least as far as we trust spike sorting). I adopted the same strategy when training the Naive Bayes.


For each feature (similarity and distance), we can make conditional distributions $$p(feature | label)$$ for each label (match and non-match) by comparing neurons to their other copies for positive pairs and comparing them to other neurons for negative pairs. 


### Understanding the issue

To investigate the poor performance, I went back to a plot that we often use for sanity-checking (credit to Célian Bimbard for the idea). We just take a single pair of sessions and organise the neurons into a matrix. Each cell in the matrix is the value of some metric for the corresponding pair of neurons. The top-left and bottom-right parts of the matrix are comparing neurons within the same session while the other two are comparing across sessions. In the ideal case, each sub-matrix would have a strong diagonal with everything off-diagonal being small. We should at least expect this to be the the case for the main diagonal of the full matrix, as neurons are generally more similar to themselves than to other neurons. We can see this for the neural net similarity matrix:


<img src="{{ '/assets/images/posts/dnnsim.png' | relative_url }}" alt="Suyash Agarwal" style="width: 50%;">

The main diagonal is strong, with weaker sub-diagonals for the two across-session sub-matrices. This gives us some indication that this method will perform well at matching neurons. But for the Naive Bayes match probabilities:

<img src="{{ '/assets/images/posts/NBProb.png' | relative_url }}" alt="Suyash Agarwal" style="width: 50%;">

We have large off-diagonal match probabilities for the same-session sub-matrices. This is despite explicitly labelling these as *different* neurons during training. So why is this happening?




