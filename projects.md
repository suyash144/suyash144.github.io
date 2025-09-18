---
layout: default
title: Projects
---

## Current and Recent Projects

### DeepUnitMatch
A neural network method for tracking neurons via their electrophysiological spike waveforms. This uses contrastive learning and a spatio-temporal autoencoder architecture to beat current state-of-the-art methods. We are in the process of writing this up!
<img src="{{ '/assets/images/DeepUnitMatch.png' | relative_url }}" alt="DeepUnitMatch" style="width: 100%; margin-bottom: 15px; margin-top: 10px;">

<img src="{{ '/assets/images/alfred-logo.png' | relative_url }}" alt="Alfred" style="float: right; width: 150px; margin-left: 20px; border-radius: 10%;">

### Alfred
An LLM-powered tool for automatic data analysis, used by scientists at UCL, Google DeepMind and the International Brain Lab, as well as professionals across industries such as consulting and finance. You can [try it out here](https://alfredthescientist.com/) or read a [short post explaining how to use it](https://suyash144.github.io/ai/research/data%20science/2025/05/27/alfred.html). The central idea is to keep the human in the loop and get AI to speed up all the boring bits of your data analysis so that you can focus on the interesting parts. You don't need any programming knowledge to gain from using the tool, and you get to decide how close to the code you want to be.

I am the sole developer on this project, built using Python and Flask for the backend and React and NodeJS for the frontend. The currently supported LLMs are Claude 4 Sonnet, Gemini 2.5 Pro, GPT-4.1 and o1.

### Suite3D
I am working on a data curation tool for volumetric cell imaging data acquired via [Suite3D](https://www.biorxiv.org/content/10.1101/2025.03.26.645628v1). Here's a screenshot to show some work in progress. The goal is to have a generalisable method of generating a latent space that captures the 3D image features. We can then do a 2D projection from this latent space (done via PCA and then UMAP in the image below) to get something which users can visualise. They can then interact with the data, applying clustering and classifying each cluster of ROIs as cells/not cells. The image shown is of the user interface I created, which allows scientists to do very fast data-driven curation that keeps the human in the loop.

<img src="{{ '/assets/images/suite3d_full.png' | relative_url }}" alt="UMAP curation tool" style="width: 60%; margin-left: auto; margin-right: auto;">

### Bayesian Radar Cosplace
My Masters' research project at the Oxford Robotics Institute, skillfully supervised by Matt Gadd and Paul Newman. We published an article on the project [here](https://doi.org/10.1049/rsn2.70002).

<img src="{{ '/assets/images/bayesian-radar-cosplace.jpg' | relative_url }}" alt="Bayesian Radar Cosplace" style="width: 100%;">
