---
layout: default # Or a specific 'post' layout if you create one later
title: "Control theory of LLMs"
date: 2025-06-18 15:00:00 +0100 # Date and time of publication
author: "Suyash Agarwal" # Optional
categories: [research, AI, control, safety] # Optional
tags: [] # Optional
---

## Appyling control theory to LLMs


A lot of exciting work is being done in AI safety at the moment, both at private companies and research labs around the world. Lately I have been thinking about applying ideas from control theory to LLMs, and wondering (with little certainty) whether this could be a way to look at AI safety research.

### What is control theory?

Control theory is a branch of engineering and applied mathematics that deals with dynamical systems and getting some desired behaviour out of them. This usually means developing a model or algorithm to steer a system towards a desired state. The reason why I think now could be a good time to bring together control theory and LLMs is that LLMs are increasingly being used as components within larger software systems (see [Alfred](http://alfredthescientist.com/) for an example I made). This is not a new observation - the authors of [this paper](https://arxiv.org/abs/2310.04444) are well aware of this motivation for developing a way to inform the inputs and control the outputs of LLMs. 

<img src="{{ '/assets/images/posts/llm-control.png' | relative_url }}" alt="LLMs as control systems" style="width: 100%;">

This figure is taken directly from their paper, called _What's the Magic Word? A Control Theory of LLM Prompting_. The problem is nicely set out in this diagram. It shoes a control-theoretic approach to LLM prompt engineering, where the goal is to determine an appropriate control input $u$ that can steer the LLM to generate an output $y$ given the initial state $x_0$. One of the most important parameters in this problem is the length of the control input seqeunce, as tokens cost compute and therefore money. This is the type of problem that optimal control is designed to solve. The effect of varying the length of $u$ is shown in the right diagram - each length corresponds to a different reachable set, which is the set of output sequences $y$ for which there exists a control input sequence $u$ that steers the LLM from initial state $x_0$ to $y$. 


To me, the problem of AI safety, in the way it is generally tackled at the moment, resembles a control system with internal states that are mostly unobservable (we don't know what the model is doing) and uncontrollable (we can't control what the model outputs). These issues are respectively tackled by research strands in interpretability and alignment, but my instinct is that this is insufficient for a few reasons:
- To extend the control theory analogy, it seems like a promising thing to do would be to bring interpretability and alignment together into one framework. I'm not sure exactly what this would look like but perhaps some inspiration can be taken from control. We wouldn't try to implement a control system for a nuclear reactor (alignment) without sensors telling us what's going on inside (interpretability). Perhaps this could offer a way around alignment faking, for example - probing alignment while simultaneously monitoring known internal circuits.
- Of course, boiling AI safety down to a control theory problem is a massive oversimplification. However, there are more practical reasons why current methods may not be sufficient. Current interpretability methods struggle to scale up to the largest, most capable models. This is a big problem - we can learn a lot from model organisms but the biggest risks are posed by the largest models via behaviours that smaller models cannot replicate.

