---
layout: default # Or a specific 'post' layout if you create one later
title: "Control theory of LLMs"
date: 2025-06-18 15:00:00 +0100 # Date and time of publication
author: "Suyash Agarwal" # Optional
categories: [research, AI, control, safety] # Optional
tags: [] # Optional
---

## Appyling control theory to LLMs


A lot of exciting work is being done in AI safety at the moment, both at private companies and research labs around the world.[One paper](https://arxiv.org/abs/2310.04444) in particular caught my eye recently, though at face value it doesn't directly relate to safety and the authors make no mention of safety in the text. 

### What is control theory?

Control theory is a branch of engineering and applied mathematics that deals with dynamical systems and getting some desired behaviour out of them. This usually means developing a model or algorithm to steer a system towards a desired state. The reason why I think now could be a good time to bring together control theory and LLMs is that LLMs are increasingly being used as components within larger software systems (see [Alfred](http://alfredthescientist.com/) for an example I made). This is not a new observation - the authors of the paper I mentioned at the beginning of this post are well aware of this motivation for developing a way to inform the inputs and control the outputs of this LLM system. 

<img src="{{ '/assets/images/posts/res.png' | relative_url }}" alt="LLMs as control systems" style="width: 100%;">



To me, the problem of AI safety, in the way it is generally tackled at the moment, resembles a control system with internal states that are mostly unobservable (we don't know what the model is doing) and uncontrollable (we can't control what the model outputs). These issues are respectively tackled by research strands in interpretability and alignment, but my instinct is that this is insufficient for a few reasons:
- To extend the control theory analogy, it seems like a promising thing to do would be to bring interpretability and alignment together into one framework. I'm not sure exactly what this would look like but perhaps some inspiration can be taken from control. We wouldn't try to implement a control system for a nuclear reactor (alignment) without sensors telling us what's going on inside (interpretability). Perhaps this could offer a way around alignment faking, for example - probing alignment while simultaneously monitoring known internal circuits.
- Of course, boiling AI safety down to a control theory problem is a massive oversimplification. However, there are more practical reasons why current methods may not be sufficient. Current interpretability methods struggle to scale up to the largest, most capable models. This is a big problem - we can learn a lot from model organisms but the biggest risks are posed by the largest models via behaviours that smaller models cannot replicate.
- Moving away from the technical challenge of controlling an AI system, perhaps the biggest risk is posed by what _people_ can do once they get their hands on powerful open-source LLMs. Any alignment research then goes out the window - we could have a perfect recipe for alignment but it's only useful if the person training their model refers to it. To me, this is the biggest reason why current research efforts, while very impressive, are simply not enough to address the risks posed by AI. 

I should say at this point that I completely understand why AI safety research has gone in the direction of controlling AI outputs and understanding how models work. These are technical, definable questions to ask and we can use research techniques from machine learning (or in some cases, neuroscience) to try to answer them. I also don't mean to minimse the fascinating research that is being done already. I think Anthropic's recent paper on [tracing the thoughts of a large language model](https://transformer-circuits.pub/2025/attribution-graphs/biology.html) was one of the most interesting papers I've ever read, especially as someone caught between the worlds of deep learning and neuroscience. But in the next section I'll discuss why I don't think this is even close to being sufficient.


### The risks

Although I said I would focus on the last question, it's hard to answer that without briefly touching on the risks posed by AI. Here's a brief non-exhaustive list of some scenarios, ~~some~~all of which are already playing out:
- jobs get replaced by AI en masse, leading to high unemployment and income inequality
- people stop thinking critically altogether
- malicious actors use AI to influence elections or spread propaganda
- the internet gets filled up with AI slop and/or misinformation
- creative industries fail as the arts get undervalued by the proliferation of AI-generated art, music and text

I'm sure I will have missed some things off this list but these seem to be the key immediate risks to society. I'm deliberately avoiding calling out the 'existential' risk of superintelligent AI, as it's a subject of debate and feels a little speculative at the moment. These are real issues that are already happening. 

### The disconnect 

The way I see it, there is a disconnect between existing AI research and the risks outlined above. Existing research tackles problems such as alignment, interpretability, scalable oversight and robustness to adversarial/anomalous inputs. 

But the real issue is deeper - we have to plan for a future where anyone can train their own LLM to do whatever they want it to. Studying model alignment is only useful as long as the individuals and companies training AI models commit to aligning their models with 'good' human values. 



