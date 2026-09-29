---
layout: post
title: "When Beliefs Bias LLM Reasoning"
date: 2026-09-08
categories: llm-behavior
tags: [behavioural-science, llms, reasoning]
card_title: When Beliefs Bias LLM Reasoning
card_description: "Can a model follow an argument's logic when its conclusion sounds wrong? I test whether familiar beliefs get in the way."
related_posts: false
description: "A behavioral demonstration of belief bias in language-model reasoning."
---

When we judge an argument, what we already believe can get in the way. We may
accept a conclusion because it sounds true, or reject it because it sounds
false, instead of checking whether it follows from the information given.
This tendency is called **belief bias**
([Evans, Barston, & Pollard, 1983](https://doi.org/10.3758/BF03196976)).
Here, I explore whether an LLM shows a similar pattern.

To test this, I give the model a few statements and ask whether a conclusion
follows from them. Some problems use familiar facts, others conflict with
everyday knowledge, and others use abstract letters such as A, B, and C. Each
has a known correct answer, allowing me to check whether the model's accuracy
changes with the content.

The problems come from **NeuBAROCO**, a published collection adapted from
reasoning tasks used in human research. It contains English and Japanese
problems, the correct answers, and labels that help researchers study different
reasoning biases
([Ando et al., 2023](https://aclanthology.org/2023.naloma-1.1/);
[Ozeki et al., 2024](https://aclanthology.org/2024.findings-acl.950/)).

> **This is a worked example of an LLM experiment.** I use a published task
> with a different model and repeat the questions at several response settings.
> The aim is to show how to run and examine the experiment. Similar answers
> from a model and a person would not, by themselves, show that they reason
> in the same way.

## The task separates logic from familiar content

Each problem contains two or three premises and a hypothesis. The model must
classify their logical relationship as:

- **entailment:** the hypothesis follows from the premises;
- **contradiction:** the hypothesis is incompatible with the premises; or
- **neither:** the premises neither entail nor contradict the hypothesis.

For example:

```text
Premise 1: Some A are B.
Premise 2: All B are C.
Hypothesis: Some A are C.
```

The correct answer is `entailment`. Because A, B, and C have no familiar
meaning, ordinary knowledge cannot make the conclusion sound more or less
plausible.

NeuBAROCO uses four content classifications:

| Content classification | What it means                                                  | Included in analysis |
| ---------------------- | -------------------------------------------------------------- | -------------------- |
| Belief-consistent      | The premises and hypothesis agree with ordinary knowledge      | 158                  |
| Belief-inconsistent    | At least one statement conflicts with ordinary knowledge       | 102                  |
| Symbolic               | The terms are abstract letters rather than familiar categories | 95                   |
| Other                  | The belief classification was unclear                          | 11                   |

The central comparison is between the first three groups. The 11 problems
classified as _other_ remain in the complete dataset and results, but they are
not used to interpret belief consistency.

## The analysis includes 366 published problems

The model completed all 375 English problems in the published NeuBAROCO NALOMA
file. During the material audit, we found that nine problems contained a third
premise that was omitted by the accompanying prompt-building code. Without that
premise, the published hypothesis could no longer be classified as intended.
Our collection retained the third premise, which corrected the omission but
departed from the published implementation. We therefore excluded these nine
problems from the analysis, leaving 366. Their responses remain in the raw
record.

Every included problem was presented three times at each temperature from 0.0
to 1.0, in steps of 0.1:

```text
366 problems × 3 repetitions × 11 temperatures = 12,078 responses
```

The model was `Qwen/Qwen2.5-3B-Instruct`, served locally with vLLM 0.22.1.
`top_p` remained fixed at 1.0 and the maximum response length was 16 tokens.
Only temperature changed between runs.

Temperature controls how much randomness is introduced when the model selects
its next token. Testing the full range therefore answers two separate
questions: does the content pattern remain visible across generation settings,
and does the model give the same answer when an identical problem is repeated?

The source file, model settings, randomization seed, and a checksum for the
trial schedule were recorded for the original collection. The same schedule
was used at every temperature, allowing each problem to be compared with itself
across settings. The nine-item exclusion was added during the post-collection
audit and is identified as such in the final analysis record.

## The published prompt and scoring rule were retained

The model received the English zero-shot instructions used in the NeuBAROCO
implementation. Each prompt followed this structure:

```text
Determine the correct logical relationship between the given premises and
the hypothesis.

Answer "entailment" if the hypothesis follows logically from the premises.
Answer "contradiction" if the premises and hypothesis are logically
incompatible. Answer "neither" if the relationship is neither entailment
nor contradiction.

Premise 1: ...
Premise 2: ...
Hypothesis: ...
The answer is:
```

The model was instructed to return one word: `entailment`, `contradiction`, or
`neither`. The dataset label `neutral` was shown as `neither`, following the
published prompt.

The primary scoring rule also follows the published implementation. It looks
at the final line of the response and checks its first word. A second, stricter
rule accepts a response only when the complete response contains exactly one
of the three labels. Both rules produced the same scores in the completed run.

One response, at temperature 0.9, did not contain a valid answer before reaching
the 16-token limit. It was retained and marked invalid. It was not silently
repaired or sent to the model again.

## Build and inspect the schedule before running the model

The scripts and source audit are available in
[`llm-behavior-experiments`](https://github.com/marishkamehta/llm-behavior-experiments). The NeuBAROCO materials are
released under CC BY 4.0 in the authors'
[official repository](https://github.com/kmineshima/NeuBAROCO).

After downloading the pinned source described in the demonstration README,
build the complete schedule from the [`llm-behavior-experiments`](https://github.com/marishkamehta/llm-behavior-experiments)
folder:

```bash
python -m belief_bias_demo.build_schedule \
  --source SOURCE/acl2024/NeuBAROCO_NALOMA.tsv \
  --output-dir OUTPUT \
  --seed 20260816 \
  --mode full \
  --exclude-three-premise-items
```

Replace `SOURCE` with the location of the downloaded NeuBAROCO repository and
`OUTPUT` with the folder where the schedule should be saved. The builder checks
the source file before creating the schedule and stops if the file does not
match the version used for this demonstration.

To collect the same temperature series, run:

```bash
python -m belief_bias_demo.run_temperature_series \
  --schedule OUTPUT/trials.jsonl \
  --build-manifest OUTPUT/build-manifest.json \
  --runs-root RUNS \
  --model qwen-3b \
  --max-tokens 16
```

The model name must match an entry in `model-registry.yaml`. A different local
or hosted model can be substituted, but its results will describe a different
model environment. The complete series requires 12,078 calls. The repository
also includes scripts for parsing each run and combining the results across
temperatures.

## Belief-inconsistent problems were harder for Qwen

Across the eleven temperatures, Qwen's accuracy was consistently lowest for
belief-inconsistent problems. Accuracy ranged from 52.9% to 57.2% for these
problems, compared with 67.1% to 70.3% for belief-consistent problems and 70.5%
to 74.4% for symbolic problems.

{% include figure.liquid
  path="assets/img/blog/belief-bias-accuracy.svg"
  alt="Qwen's accuracy was lower for belief-inconsistent NeuBAROCO problems at every tested temperature."
  caption="Figure 1. Qwen's accuracy was lower for belief-inconsistent NeuBAROCO problems at every tested temperature."
  loading="lazy"
%}

The difference between belief-consistent and belief-inconsistent problems
ranged from 10.5 to 16.7 percentage points. This is the qualitative pattern the
demonstration was designed to detect: Qwen made more classification errors when
the logical problem contained statements that conflicted with ordinary belief.

Ando et al. also reported the lowest accuracy on belief-inconsistent problems
for every model they evaluated. Their study provides the source task and the
comparison pattern; the percentages should not be compared directly because
the models and generation procedures differ.

## Temperature changed repeatability more than accuracy

Overall accuracy remained between 65.0% and 66.7% across the full temperature
range. There was no clear monotonic improvement or decline as temperature
increased. The gap between belief-consistent and belief-inconsistent problems
also remained at every setting.

The repeated responses became less consistent. **Repetition stability**
measures how often the three responses to the same problem agreed.
Mean stability declined from 99.7% at temperature 0.0 to 94.2% at temperature
1.0.

{% include figure.liquid
  path="assets/img/blog/belief-bias-temperature.svg"
  alt="Qwen's overall accuracy changed little across temperatures, while repeated answers became less consistent."
  caption="Figure 2. Qwen's overall accuracy changed little across temperatures, while repeated answers became less consistent."
  loading="lazy"
%}

This distinction matters experimentally. Similar average accuracy can conceal
a change in how reliably the model returns the same answer. Temperature was
therefore part of the experimental condition, not merely a deployment detail.

## What this demonstration shows—and what it does not

Under these conditions, Qwen's logical classifications varied with the content
of the problem. This does not show that Qwen holds beliefs, experiences
conflict, or uses the same cognitive mechanisms as a person. Explaining why
humans and models might produce a similar pattern would require a different
study.

The code, source audit, aggregate analysis tables, parser checks, and figures
are available in
[`llm-behavior-experiments`](https://github.com/marishkamehta/llm-behavior-experiments).
Raw model responses and generated trial schedules are not bundled. Recreating
the reported results requires retrieving the pinned materials, building the
schedule, and collecting or obtaining the underlying responses.

---

[← All blog articles](/blog/) ·
[LLM experiment guides](/blog/getting-started/)
