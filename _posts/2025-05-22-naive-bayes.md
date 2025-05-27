---
layout: default # Or a specific 'post' layout if you create one later
title: "Naive Bayes naivety"
date: 2025-05-23 15:00:00 +0100 # Date and time of publication
author: "Suyash Agarwal" # Optional
categories: [machine learning, research] # Optional
tags: [statistics] # Optional
---

## Naive Bayes naivety

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

### My first attempt at Naive Bayes

The way we had incorporated the spatial information before was to take the matches predicted by the neural network and then remove the ones that are spatially distant from each other. But we wanted to try something a bit smarter, that could take both waveform similarity and spatial distance into account at the same time, rather than having these two steps in series.

To this end, we thought a Naive Bayes could work with the neural network similarity and distance between two neurons as its two features. Here, each sample is a pair of neurons. I trained one of these using a kernel density estimator to get conditional distributions for each feature (conditioned on our two labels; match and non-match). All seemed well and I expected this to do at least as well as the neural network with spatial filtering and *much* better than the neural network alone. But...

<img src="{{ '/assets/images/posts/res.png' | relative_url }}" alt="Naive Bayes vs original method" style="width: 100%;">

To explain this, the left plot is AUC (an accuracy score between 0 and 1) and the right plot is the number of matches found. The two axes are the original method (neural network similarity scores followed by spatial filtering on the x-axis) and the new Naive Bayes method (on the y-axis). Each point represents a pair of recording sessions on which our two performance metrics were computed. You can see that the Naive Bayes is not performing as well as the original method. What if we compare it to the original method without any spatial filtering? If the Naive Bayes doesn't win that comparison then something has to be wrong.

<img src="{{ '/assets/images/posts/res2.png' | relative_url }}" alt="Naive Bayes vs original method" style="width: 100%;">

Hmm. The Naive Bayes (which has both waveform similarity and spatial information to work with) has a similar AUC to a method that only looks at waveform similarity. The latter also finds *more* matches, though this may just be because the spatial filtering step acts only to reduce the number of matches returned.


### Training without labels: an aside

Before we can understand what is going wrong here, I need to detail how exactly I trained the Naive Bayes. This requires labelled data, which is something that we always struggle with in this project. We never actually have ground truth neuron labels, so the machine learning methods we could use were either self-supervised (training the autoencoder) or they used a quasi-ground truth that we could construct from the fact that within a session, we know which spikes come from which neuron thanks to upstream spike-sorting software. We also have two copies of each neuron's waveform (averaged over the first and second half of the session). This means we can do contrastive learning if we train only on within-session data, because we have pairs of neurons which are the same and pair of neurons which are different (at least as far as we trust spike sorting). I adopted the same strategy when training the Naive Bayes.


For each feature (similarity and distance), we can make conditional distributions $p(\text{feature} \mid \text{label})$ for each label (match and non-match) by comparing neurons to their other copies for match pairs and comparing them to other neurons for non-match pairs. 


### Understanding the issue

To investigate the poor performance, I went back to a plot that we often use for sanity-checking. We just take a single pair of sessions and organise the neurons into a matrix. Each cell in the matrix is the value of some metric for the corresponding pair of neurons. The top-left and bottom-right parts of the matrix are comparing neurons within the same session while the other two are comparing across sessions. In the ideal case, each of the four sub-matrices would have a strong diagonal with everything off-diagonal being small. We should at least expect this to be the the case for the main diagonal of the full matrix, as neurons are generally more similar to themselves than to other neurons. We can see this for the neural net similarity matrix:


<img src="{{ '/assets/images/posts/dnnsim.png' | relative_url }}" alt="Waveform similarity matrix" style="display: block; width: 50%; margin-left: auto; margin-right: auto;">

The main diagonal is strong, with weaker sub-diagonals for the two across-session sub-matrices. This gives us some indication that this method will perform well at matching neurons, as:
- We have a metric that confirms that neurons are more similar to themselves than to other neurons.
- The metric is also able to identify potential matches across sessions.

 But for the Naive Bayes match probabilities:

<img src="{{ '/assets/images/posts/NBProb.png' | relative_url }}" alt="Naive Bayes output" style="display: block; width: 50%; margin-left: auto; margin-right: auto;">

We have large off-diagonal match probabilities for the same-session sub-matrices. This is despite explicitly labelling these as *different* neurons during training. So why is this happening?

The Naive Bayes has two features to work with: waveform similarity and spatial distance. We've seen the waveform similarity matrix and it looks fine. It can't explain the patterns we observe in the Naive Bayes output so let's look at the same matrix with distance.

<img src="{{ '/assets/images/posts/distance.png' | relative_url }}" alt="Distance matrix" style="display: block; width: 50%; margin-left: auto; margin-right: auto;">

Hard to read anything from this. But we can see that the scale goes up to over 2500um. This is after drift correction (where we correct for the overall shift in electrode position between the two sessions) so any pair of neurons with a distance over around 150um is highly unlikely to be the same neuron. We can just focus on the pairs that have a chance of being the same neuron by clipping the values at 150um:

<img src="{{ '/assets/images/posts/distance_clipped.png' | relative_url }}" alt="Distance matrix" style="display: block; width: 50%; margin-left: auto; margin-right: auto;">

Now everything at 150um and over is white. And we see the same patterns we saw in the Naive Bayes output matrix! So it looks like the Naive Bayes is assigning high match probability to neurons that have 'low' spatial distance, even though it was told during training that these are not matches.
This is in contrast to the neural network output, where these same pairs generally have low waveform similarity.

Effectively, the Naive Bayes is being given somewhat contradictory information. It is trained to assign high match probabilities when distances are low and waveform similarities are high. For these problematic within-session pairs, similarity is low but distance is also relatively low. This is where the range of distances that we include in training data becomes a problem. 150um is actually still a fairly high distance; it just seems low compared to 2500um. In fact, anything over 150um is almost guaranteed not to be a match so the vast majority (over 90%) of the distance 'signal' tells us nothing.

So given contradictory information (low distance *and* low waveform similarity simultaneously), the question is which feature will the Naive Bayes listen to more?

### A tiny bit of maths

From a training point of view, the Naive Bayes *by construction* learns that every neuron that is a match (which we define as on-diagonal, within-session neurons, ie the spike-sorting output) has 0 distance. Likewise, most non-match pairs have non-zero distance (different neurons in the same session are unlikely to be in the same spatial location). So from the perspective of the Naive Bayes, it can achieve near-perfect classification accuracy by only looking at distance! Waveform similarity on the other hand is a much noisier signal so will be weighted less as a feature.
This is at odds with what we know from earlier work in the project, which is that:
- The neural network similarity is a good classifier, it just doesn’t have distance information.
- Distance alone is not a good classifier (see the distance matrices).

I will try and give some hand-wavey non-rigorous mathematical intuition for this. I have been told that no one will read these things if I stuff them full of equations so I will limit myself as much as possible. Luckily, the [Wikipedia article](https://en.wikipedia.org/wiki/Naive_Bayes_classifier) does a pretty good job of explaining the conditional independence assumption and how the classifier works so I won't rehash that here. Assuming you're up to speed on that, the Naive Bayes will compute the following match probability for a pair of neurons that have waveform similarity $s$ and spatial distance $d$ (this is deliberately vague, hence the lack of rigour in this section):

$$p(\text{match} \mid \text{similarity}=s, \text{distance}=d) \propto p(\text{match}) p(s \mid \text{match}) p(d \mid \text{match})$$

Here, we can see the 'naive' assumption of the Naive Bayes coming into play as we can simply multiply the conditional probabilities $p(s \mid \text{match})$ and $p(d \mid \text{match})$ to get the joint conditional $p(s, d \mid \text{match})$. The proportionality constant is $p(s, d)$ as per Bayes' rule but we omit it here as it's not relevant since we only care about the relative magnitudes of $p(\text{match} \mid \text{similarity}=s, \text{distance}=d)$ and $p(\text{non-match} \mid \text{similarity}=s, \text{distance}=d)$. After normalising, these have to sum to 1 anyway so we only need to calculate match probability.

During training, we basically set $p(d \mid \text{match}) \approx \delta(0)$ as described above. So the Naive Bayes can *almost never* assign non-zero match probability to a pair of neurons that are spatially distant. Because the conditional distribution of distance values fed into the Naive Bayes was close to a delta function (in that it is non-zero only when the distance is zero and zero almost everywhere else) and this is not the case for waveform similarity, the whole classifier totally overfits to distance. This is despite waveform similarity actually being a more informative feature.
The neural network is constrained to fit some nonlinear transformation to the actual waveforms themselves so it can't align with the spike-sorting output perfectly, hence the apparent 'noise' in that signal.

### The fix

I simply re-trained the Naive Bayes, this time using a noisier spatial distance metric. I call this method NBProbCentroid below:
<img src="{{ '/assets/images/posts/res3.png' | relative_url }}" alt="Naive Bayes vs original method" style="width: 100%;">

Now that the distance feature's conditional distribution is not a delta function (due to the noise), the Naive Bayes performs much better than before!

### Takeaways

If you read this far, I hope you gained something from this post. I just wanted to try this out as a way to remember what I've done while working on cool projects, and to practice my writing skills. The Naive Bayes is a great tool to be able to use, but on this occasion there were some pitfalls that were not obvious to me at the start. As well as being aware of the conditional independence assumption, make sure your feature distributions are reasonable!

