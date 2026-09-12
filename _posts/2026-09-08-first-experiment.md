---
layout: post
title: "Quick Start Guide II: A First Experiment with Silico"
date: 2026-09-08
categories: llm-behavior
description: "A first controlled behavioral experiment with an LLM using Silico."
---

In [Quick Start Guide I](/blog/2026/quick-start/), you created a free-tier Google AI
Studio API key and sent a message to a model. Now you will reuse that key to run
a small behavioral experiment.

The experiment changes one feature of a choice while holding the original

response, and compare the two conditions. This is the basic structure of a
controlled experiment.

We will use `silico`, a small Python library for behavioral experiments with
LLMs. [Silico](SILICO_REPOSITORY_URL) provides a common interface for hosted
and self-hosted models and represents conversations as turns and trajectories.

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

| Hotel                     | Guest rating | Price | Role                                     |
| ------------------------- | ------------ | ----- | ---------------------------------------- |
| A                         | 8.5/10       | $220  | Target                                   |
| B                         | 7.5/10       | $130  | Competitor                               |
| C                         | 8.3/10       | $235  | Decoy, shown only in the decoy condition |
| {: .table-hotel-options } |

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
git clone SILICO_REPOSITORY_URL
cd llm-behavior
python -m venv .venv --prompt llm-behavior
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

This reuses the `GEMINI_API_KEY` environment variable from Quick Start I. The
registry reads the key from the environment, so it does not appear in your code
or configuration file.

## Run the experiment

Create a file named `decoy_experiment.py` at the project root:

```python
import csv
import time
from collections import Counter
from pathlib import Path

import requests
from silico.registry import make_llm


llm = make_llm("gemini-flash")

COMMON = (
    "You are booking a hotel for one night. The hotels differ only in guest "
    "rating and price. Choose one hotel. Reply with only its letter.\n\n"
    "A: Guest rating 8.5 out of 10; price $220.\n"
    "B: Guest rating 7.5 out of 10; price $130."
)

PROMPTS = {
    "control": COMMON,
    "decoy": COMMON + "\nC: Guest rating 8.3 out of 10; price $235.",
}
SCHEDULE = ["control", "decoy"] * 5
RESULTS = Path("decoy_results.csv")

# Resume after the last complete row if an earlier run was interrupted.
completed = 0
if RESULTS.exists():
    with RESULTS.open(newline="", encoding="utf-8") as existing_file:
        completed = sum(1 for _ in csv.DictReader(existing_file))

mode = "a" if completed else "w"
with RESULTS.open(mode, newline="", encoding="utf-8") as output:
    writer = csv.DictWriter(
        output,
        fieldnames=["observation", "condition", "prompt", "raw_response", "choice"],
    )
    if not completed:
        writer.writeheader()

    for observation, condition in list(enumerate(SCHEDULE, start=1))[completed:]:
        for attempt in range(1, 4):
            try:
                raw_response = llm.respond(PROMPTS[condition])
                break
            except (requests.ReadTimeout, requests.HTTPError) as error:
                retryable = isinstance(error, requests.ReadTimeout) or (
                    error.response.status_code == 503
                )
                if not retryable or attempt == 3:
                    raise
                wait_seconds = 15 * attempt
                print(f"Temporary error; retrying in {wait_seconds} seconds.")
                time.sleep(wait_seconds)

        normalized = raw_response.strip().upper()
        choice = normalized if normalized in {"A", "B", "C"} else "INVALID"
        writer.writerow(
            {
                "observation": observation,
                "condition": condition,
                "prompt": PROMPTS[condition],
                "raw_response": raw_response,
                "choice": choice,
            }
        )
        output.flush()
        print(condition, choice)

with RESULTS.open(newline="", encoding="utf-8") as results_file:
    rows = list(csv.DictReader(results_file))

print("\nSummary")
for condition in PROMPTS:
    counts = Counter(row["choice"] for row in rows if row["condition"] == condition)
    print(condition, dict(counts))
```

Run it from the activated environment:

```bash
python decoy_experiment.py
```

The complete experiment makes ten successful requests: five in each condition.
It writes every prompt, raw response, and parsed choice to
`decoy_results.csv`, flushing the file after each observation. If a timeout or
temporary service error interrupts the run, start the script again; it reads
the existing rows and resumes at the next observation.

Check your current Gemini limits before running it. If the API reports a rate
limit, wait for the period shown in the error message before trying again.

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
  caption="Figure 1. Gemini chose hotel B on all five control observations. When inferior hotel C was added, it chose target hotel A on four observations and hotel B on one."
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
[Series contents](/blog/2026/controlled-experiments/#what-this-series-covers) ·
[Next: Infrastructure terms for behavioral researchers →](/blog/2026/infrastructure-vocabulary/)
