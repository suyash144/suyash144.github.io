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

### Why control theory?

Control theory is a branch of engineering and applied mathematics that deals with dynamical systems and getting some desired behaviour out of them. This usually means developing a model or algorithm to steer a system towards a desired state. The reason why I think now could be a good time to bring together control theory and LLMs is that LLMs are increasingly being used as components within larger software systems (see [Alfred](http://alfredthescientist.com/) for an example I made). This is not a new observation - the authors of [this paper](https://arxiv.org/abs/2310.04444) are well aware of this motivation for developing a way to inform the inputs and control the outputs of LLMs. 

<img src="{{ '/assets/images/posts/llm-control.png' | relative_url }}" alt="LLMs as control systems" style="width: 100%;">

This figure is taken directly from their paper, called _What's the Magic Word? A Control Theory of LLM Prompting_. The problem is nicely set out in this diagram. It shows a control-theoretic approach to LLM prompt engineering, where the goal is to determine an appropriate control input $u$ that can steer the LLM to generate an output $y$ given the initial state $x_0$. One of the most important parameters in this problem is the length of the control input seqeunce, as tokens cost compute and therefore money. This is the type of problem that optimal control is designed to solve. The effect of varying the length of $u$ is shown in the right diagram - each length corresponds to a different reachable set, which is the set of output sequences $y$ for which there exists a control input sequence $u$ that steers the LLM from initial state $x_0$ to $y$. 

### What does this have to do with AI safety?

Maybe nothing. But to me, the problem of AI safety resembles a control system with internal states that are mostly unobservable (we don't know what the model is doing) and uncontrollable (we can't control what the model outputs). These issues are respectively tackled by research strands in interpretability and alignment. To use some of the language of control theory, we can let $t$ index generation steps and define:
- hidden internal state $x_t$. This would be an extremely high-dimensional vector representing activations, latent intent variables, etc.
- control input $u_t$. In the context of AI safety/control, this would be an alignment intervention, e.g. injected safety prompts.
- stochastic disturbance $w_t$. This captures model noise, sampling randomness, etc. 
- output vector $y_t$.

Then we can model the LLM as a discrete-time nonlinear stochastic dynamical system:

$$x_{t+1} = f(x_t, u_t) + w_t$$

$$y_t = h(x_t) + v_t$$

Here, $f$ is unknown as it captures the transformer forward pass and some update of the latent internal variables across generation steps. $h$ maps internal state to model output with some associated measurement noise $v_t$. Then the goal is to design a controller $u_t$ which is some feedback law based on estimated state $\hat{x}_t$ and its history up to generation step $t$. The aim of this would be to keep output behaviour in a specified safe set using minimal control input tokens while being robust to model uncertainty and adversarial inputs. The fields of robust control and optimal control seem well-equipped to handle this, but I suspect the main issue with this in practice would be the huge state space of possible token sequences. 

Within this framework, interpretability (or at least some aspects of it) can be thought of as state estimation:

$$\hat{x}_{t|t} = \text{Observer}(\hat{x}_{t-1|t-1}, u_{t-1}, y_t)$$

where the estimator would be something like a Kalman filter in classic control theory. Similarly, alignment can be thought of as a model predictive control problem solved at each step over some horizon $H$. 

### Takeaways

I'm still not sure what exactly the takeaways are from this little thought experiment. If anyone reading this can point out major flaws in this idea or formulation then please get in touch! 
One conclusion that we can perhaps draw is that it seems like a good idea to bring interpretability and alignment together into one framework. I'm not sure exactly what this would look like but perhaps some inspiration can be taken from control. We wouldn't try to implement a control system for a nuclear reactor (alignment) without sensors telling us what's going on inside (interpretability). Perhaps this could offer a way around alignment faking (observed by Anthropic [here](https://www.anthropic.com/research/alignment-faking)), for example - probing alignment while simultaneously monitoring known internal circuits.
Of course, boiling AI safety down to a control theory problem is an oversimplification, but maybe some useful ideas can be transferred over. 

If you're working on either control theory or AI safety and you have some thoughts on this then I'd be super curious to hear them!

