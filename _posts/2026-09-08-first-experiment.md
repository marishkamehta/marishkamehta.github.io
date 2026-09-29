---
layout: post
title: "Testing the Decoy Effect in an LLM"
date: 2026-09-08
categories: llm-behavior
series: getting-started
related_posts: false
description: "A worked example using Silico to test whether adding a decoy changes an LLM's choices."
---

In [Quick Start Guide: Hello LLM!](/blog/2026/quick-start/), you created an API key in
Google AI Studio and used it to send a message to a model on the free tier.
Now you will reuse that key to run a small behavioral experiment with
[Silico](https://github.com/marishkamehta/silico), a Python library for behavioral experiments
with LLMs. Silico provides a common interface for hosted and self-hosted models
and helps you organize their responses for analysis.

In this worked example, you will ask the model to choose between two hotels, then add a third hotel
while keeping the original options and instructions unchanged. By recording
the model's choices across repeated requests in both conditions, you can
explore whether the added option changes its choices. This is a simple example
of a controlled experiment.

## Test whether a decoy changes a choice

The **decoy effect** occurs when adding an inferior third option changes the
relative appeal of two existing options. The new option is asymmetrically
dominated: one original option is better than it on every stated attribute,
while the other original option involves a different tradeoff. This pattern
was introduced in consumer-choice research by Huber, Payne, and Puto
([1982](https://doi.org/10.1086/208899)) and has also been studied in
instruction-tuned language models ([Itzhak et al.,
2024](https://aclanthology.org/2024.tacl-1.43/)).

Our model chooses a hotel using only guest rating and price:

<table class="table-hotel-options">
  <thead>
    <tr>
      <th scope="col">Hotel</th>
      <th scope="col">Guest rating</th>
      <th scope="col">Price</th>
      <th scope="col">Role</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>A</td>
      <td>8.5/10</td>
      <td>$220</td>
      <td>Target</td>
    </tr>
    <tr>
      <td>B</td>
      <td>7.5/10</td>
      <td>$130</td>
      <td>Competitor</td>
    </tr>
    <tr>
      <td>C</td>
      <td>8.3/10</td>
      <td>$235</td>
      <td>Decoy, shown only in the decoy condition</td>
    </tr>
  </tbody>
</table>

Hotel C is worse than A on both attributes: it has a lower rating and a higher
price. It is not similarly dominated by B because B is cheaper but also has a
lower rating. A reader might therefore expect C to be irrelevant. The
experiment tests whether its presence changes the model's choice between A and
B.

This demonstration measures an observable response pattern. It does not imply
that a language model and a person make choices through the same mechanisms.

## Set up Silico

Clone the repository, create a virtual environment, and install the package:

```bash
git clone https://github.com/marishkamehta/silico
cd silico
python -m venv .venv --prompt silico
source .venv/bin/activate
pip install -e .
```

Add the hosted Gemini model to `model-registry.yaml` at the project root:

```yaml
gemini-flash:
  backend: gemini
  base_url: https://generativelanguage.googleapis.com/v1beta/openai
  served_name: gemini-3.6-flash
  key_env: GEMINI_API_KEY
```

This reuses the `GEMINI_API_KEY` environment variable from the quick start guide. The
registry reads the key from the environment, so it does not appear in your code
or configuration file.

## Run the experiment

The core experiment uses two prompts. The control offers hotels A and B;
the decoy condition adds hotel C while keeping everything else the same.

```python
from silico.registry import make_llm

llm = make_llm("gemini-flash")

CONTROL_PROMPT = (
    "You are booking a hotel for one night. "
    "The hotels differ only in guest rating and price. "
    "Choose one hotel. Reply with only its letter.\n\n"
    "A: Guest rating 8.5 out of 10; price $220.\n"
    "B: Guest rating 7.5 out of 10; price $130."
)

PROMPTS = {
    "control": CONTROL_PROMPT,
    "decoy": CONTROL_PROMPT + "\nC: Guest rating 8.3 out of 10; price $235.",
}

# Alternate conditions, making five requests for each.
for condition in ["control", "decoy"] * 5:
    raw_response = llm.respond(PROMPTS[condition])
    normalized = raw_response.strip().upper()
    choice = normalized if normalized in {"A", "B", "C"} else "INVALID"
    print(condition, choice)
```

Each request prints the condition and the model's choice. Compare how often
it chooses A with and without the decoy. Responses other than a single A, B,
or C are marked `INVALID`.

For the complete runnable version, use the
[decoy experiment script](https://github.com/marishkamehta/llm-behavior-experiments/blob/main/examples/decoy_experiment.py).
It also saves prompts and responses to `decoy_results.csv`, summarizes the
choices, retries timeouts and temporary service errors, and resumes an
interrupted run from its saved results.

## What the model sees and replies

Can adding a hotel that is worse than A change the model's choice between A
and B? The dialogue below illustrates the comparison. These are separate
requests, with prompts shortened for readability and example replies that
may vary when you run the experiment.

<figure class="experiment-dialogue mathjax_ignore" aria-labelledby="dialogue-caption">
  <div class="experiment-dialogue-grid">
    <section class="experiment-dialogue-condition" aria-labelledby="dialogue-control">
      <h3 id="dialogue-control">Without C <span>A and B</span></h3>
      <div class="experiment-message experiment-message-user">
        <p class="experiment-message-label">You ask</p>
        <p>Choose a hotel for one night based on rating and price. Reply with its letter.</p>
        <p><strong>A</strong> · 8.5/10 · <span class="experiment-price">$220</span><br><strong>B</strong> · 7.5/10 · <span class="experiment-price">$130</span></p>
      </div>
      <div class="experiment-message experiment-message-model">
        <p class="experiment-message-label">Model replies</p>
        <p class="experiment-choice">B</p>
      </div>
    </section>
    <section class="experiment-dialogue-condition" aria-labelledby="dialogue-decoy">
      <h3 id="dialogue-decoy">With C <span>Same A and B, plus C</span></h3>
      <div class="experiment-message experiment-message-user">
        <p class="experiment-message-label">You ask</p>
        <p>Choose a hotel for one night based on rating and price. Reply with its letter.</p>
        <p><strong>A</strong> · 8.5/10 · <span class="experiment-price">$220</span><br><strong>B</strong> · 7.5/10 · <span class="experiment-price">$130</span></p>
        <p class="experiment-added-option"><strong>C</strong> · 8.3/10 · <span class="experiment-price">$235</span></p>
      </div>
      <div class="experiment-message experiment-message-model">
        <p class="experiment-message-label">Model replies</p>
        <p class="experiment-choice">A</p>
      </div>
    </section>
  </div>
  <figcaption id="dialogue-caption">Figure 1. Separate exchanges with and without hotel C. Prompts are shortened and replies are illustrative; actual replies may vary.</figcaption>
</figure>

**What changed?** Only C was added. It costs more than A and has a lower rating.

**What to look for:** In this illustration, the model chooses B without C and
A with C, even though A and B are unchanged. C is never chosen, but its
presence may affect the choice.

## What happened in the tested run

We ran this procedure with `gemini-3.6-flash` through Google's
OpenAI-compatible endpoint. The model produced a valid single-letter response
on every observation:

| Condition          | Chose A | Chose B | Invalid |
| ------------------ | ------- | ------- | ------- |
| Control: A and B   | 0/5     | 5/5     | 0/5     |
| Decoy: A, B, and C | 4/5     | 1/5     | 0/5     |

{% include figure.liquid
  path="assets/img/blog/decoy-effect-results.svg"
  alt="Gemini chose hotel B on all five control observations. When inferior hotel C was added, it chose target hotel A on four observations and hotel B on one."
  caption="Figure 2. Gemini chose hotel B on all five control observations. When inferior hotel C was added, it chose target hotel A on four observations and hotel B on one."
  loading="lazy"
%}

Without the decoy, the model always selected the cheaper hotel B. After the
inferior hotel C was added, it selected hotel A on four of five observations.
The target's selection rate therefore increased from 0% to 80% in this small
run, even though A and B did not change.

This is an exploratory demonstration, not a general estimate of the decoy
effect in LLMs. The hotel values were calibrated after an initial pair made A
too attractive in both conditions. The final sample contains only five
observations per condition, uses one model and one choice problem, and was not
preregistered. A larger study would use several products, counterbalance option
labels and presentation order, fix and report generation settings, justify its
sample size, and specify its analysis in advance.

The useful lesson is procedural: the code changes one feature of the choice
set, applies the same scoring rule to both conditions, preserves raw responses,
and makes the observed contrast directly inspectable.

## What comes next

This quick start keeps the experiment small enough to inspect directly. The
rest of the series explains how to choose a backend, test a reusable pipeline,
record complete research metadata, and scale a demonstration responsibly.

---

[← Previous: Hello LLM!](/blog/2026/quick-start/) ·
[Series contents](/blog/getting-started/) ·
[Next: Infrastructure terms for behavioral researchers →](/blog/2026/infrastructure-vocabulary/)
