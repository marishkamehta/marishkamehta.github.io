---
layout: post
title: "Studying LLM Behavior: Why It Matters and How to Begin"
date: 2026-09-01
categories: llm-behavior
description: "An introduction to the experimental mindset behind studying how large language models behave under controlled conditions."
---

_Studying LLM Behavior · Introduction_

AI tools, particularly LLMs, have become part of daily life, used for both
professional and personal tasks. Imagine asking an LLM to help decide between
a beach holiday and a city break. It might favor the beach, only to recommend
the city when the options are reversed, even though the plans themselves have
not changed.

An interaction like this raises bigger questions. How stable are an LLM's
preferences? Does the order of information influence its choices? Can an
earlier exchange affect a later response? These questions connect everyday
experiences with ideas studied across psychology, behavioral science,
neuroscience, and LLM research. My research focuses on the processes underlying
human decision-making, including future-oriented planning and conflict
resolution. This makes LLM behavior interesting in its own right, while also
raising a broader question: how might interactions with LLMs shape the way
people learn, communicate, and make decisions?

To answer these questions, the conditions under which each response is produced
need to be clear and repeatable. This does not mean treating an LLM, or any AI
system, as though it were a human participant. It means drawing on more than a
century of experimental methods
developed by psychologists and behavioral scientists to test how and when an
LLM's responses change. In order to study LLM responses with the same rigour
used in human behavioral research, the experimental procedure needs to be
consistent and reproducible. This means administering the same task repeatedly,
keeping track of what the LLM has seen, introducing specific manipulations or
interventions, and saving enough information to reproduce each run.

The experimental logic felt familiar to me; putting it into practice did not.
I did not know where to begin, what I needed, or even which questions to ask.
Did I need access to a GPU, and if so, how many? Before I could answer that, I
had to familiarize myself with terms such as _model server_, _API_, _container_,
and _GPU memory_. It turns out that getting started can be much simpler. You can
get quite far with just a laptop and an internet connection.

This blog series aims to bridge that gap by showing how accessible behavioral
experiments with LLMs can be and simplifying the process of getting started. It
also aims to encourage exchange between behavioral science, neuroscience, and
LLM research, so that insights from each can inform the others.

## What this series covers

1. **[Quick Start Guide I: Hello LLM](/blog/2026/quick-start/)**
2. **[Quick Start Guide II: A First Experiment with Silico](/blog/2026/first-experiment/)**
3. **[When Beliefs Bias LLM Reasoning](/blog/2026/belief-bias/)**
4. **[First or Second: How LLMs Judge Competing Responses](/blog/2026/position-bias/)**

---

### Optional guides

These optional guides cover key concepts, infrastructure choices, and model
setup.

#### Concepts and choices

- **[Infrastructure Concepts for Behavioral Researchers](/blog/2026/infrastructure-vocabulary/)**
- **[Behavioral Experimental Concepts for Technical Readers](/blog/2026/experimental-vocabulary/)**
- **[Choosing Where the Model Runs](/blog/2026/choosing-a-backend/)**

#### Pipeline and model setup

- **[Testing the Experimental Pipeline](/blog/2026/mock-pipeline/)**
- **[Connecting Directly to a Hosted Model](/blog/2026/hosted-api/)**
- **[Running a Local Model with Ollama](/blog/2026/ollama/)**
- **[Connecting to an Existing Azure Deployment](/blog/2026/azure/)**
- **[Running a Model with vLLM](/blog/2026/vllm/)**

---

[Series contents](#what-this-series-covers) ·
[Next: Quick Start Guide →](/blog/2026/quick-start/)
