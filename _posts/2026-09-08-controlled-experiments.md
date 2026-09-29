---
layout: post
title: "Studying LLM Behavior: Why It Matters and Where to Begin"
date: 2026-09-01
categories: llm-behavior
tags: [behavioural-science, llms, getting-started]
card_description: "New to language models? Learn how to ask a research question, run a simple experiment, and understand the responses."
series: getting-started
related_posts: false
description: "An introduction to the experimental mindset behind studying how large language models behave under controlled conditions."
---

This series is for behavioral researchers interested in studying how large
language models (LLMs) respond to different tasks and how those responses change
with the information provided. The guides show how to apply methods from
behavioral research to LLMs, from designing a simple experiment to interpreting
the model's responses. I also explain the technical concepts I had to learn
along the way, so that readers new to working with LLMs have a place to begin.

How stable are an LLM's
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

The code lives in two repositories: [Silico](https://github.com/marishkamehta/silico)
provides the tools for interacting with models, while
[llm-behavior-experiments](https://github.com/marishkamehta/llm-behavior-experiments)
contains the worked experiments, source audits, and reviewed results.

## What this series covers

1. **[Quick Start Guide: Hello LLM!](/blog/2026/quick-start/)**
2. **[Testing the Decoy Effect in an LLM](/blog/2026/first-experiment/)**

The experimental demonstrations on [belief bias](/blog/2026/belief-bias/) and
[position bias](/blog/2026/position-bias/) are separate, standalone articles.

---

### Appendices

These appendices are optional references for the main series. Use Appendix A
to look up unfamiliar terms and choose where to run a model, and Appendix B
when you are ready to configure that connection and test your experiment.

#### Appendix A: Terminology and model hosting

- **[Infrastructure Concepts for Behavioral Researchers](/blog/2026/infrastructure-vocabulary/)**
- **[Behavioral Experimental Concepts for Technical Readers](/blog/2026/experimental-vocabulary/)**
- **[Choosing Where the Model Runs](/blog/2026/choosing-a-backend/)**

#### Appendix B: Model setup and pipeline testing

- **[Testing the Experimental Pipeline](/blog/2026/mock-pipeline/)**
- **[Connecting Directly to a Hosted Model](/blog/2026/hosted-api/)**
- **[Running a Local Model with Ollama](/blog/2026/ollama/)**
- **[Connecting to an Existing Azure Deployment](/blog/2026/azure/)**
- **[Running a Model with vLLM](/blog/2026/vllm/)**

---

[Series contents](/blog/getting-started/) ·
[Next: Quick Start Guide →](/blog/2026/quick-start/)
