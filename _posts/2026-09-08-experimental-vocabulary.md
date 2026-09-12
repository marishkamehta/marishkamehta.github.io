---
layout: post
title: "The Experimental Vocabulary Behind the Pipeline"
date: 2026-09-08
categories: llm-behavior
description: "Core experimental-design terms for building interpretable LLM studies."
---

_Studying LLM Behavior · Vocabulary_

Generating and collecting model responses is only part of the work. Before
those responses can be treated as data, we need to know what each one
represents and which conclusions the study can support.

Thousands of model calls do not necessarily produce thousands of independent
observations. We still need to distinguish a trial from a session, identify the
experimental unit, and determine which observations are independent. These are
decisions about experimental design, not decisions the API can make for us.

## The experiment built around the responses

| Term                         | Meaning in this series                                                                                  |
| ---------------------------- | ------------------------------------------------------------------------------------------------------- |
| Experimental unit            | The smallest unit independently assigned to a condition or intervention                                 |
| Trial                        | One structured opportunity to respond within an experiment                                              |
| Session                      | An ordered set of trials or interactions treated as belonging together                                  |
| Condition                    | A defined version of the experimental procedure                                                         |
| Manipulation or intervention | A deliberate difference introduced to test how it changes an outcome                                    |
| Outcome                      | The response feature that will be measured or scored                                                    |
| Experimental control         | Holding relevant features constant or varying them according to a defined design                        |
| Randomization                | Using a random procedure to assign units or order events                                                |
| Counterbalancing             | Systematically varying order or assignment so that it is not confounded with the condition of interest  |
| Order effect                 | A change in responses associated with when or in what sequence something is presented                   |
| Practice effect              | A change in performance associated with repeated experience of a task or measure                        |
| Repeated measure             | More than one observation obtained from the same experimental unit                                      |
| Independence                 | The assumption that one observation does not provide information about or depend on another observation |
| Exclusion criterion          | A rule specifying when an observation or unit will not enter an analysis                                |
| Replicability                | Whether a new study collecting new data obtains results consistent with an earlier study                |
| Generalizability             | The extent to which a result applies beyond the models, items, settings, or contexts directly studied   |

The experimental unit deserves particular care. It cannot be inferred simply
by counting API requests. Three requests to the same model configuration may
represent repeated observations within one session, three separate sessions,
or three independently defined experimental units, depending on the design.
Calling them three participants does not make them independent.

The same caution applies to a trial. A model call can contain one trial, but it
can also contain an entire multi-trial conversation, or one trial plus the full
history of everything that came before it. The unit sent through the API and
the unit used in the analysis are related, but they are not automatically the
same.

{% include figure.liquid
  path="assets/img/blog/experiment-units.svg"
  alt="A study assigns experimental units to conditions; each unit may complete a session containing several trials, while each trial may generate one or more technical API calls."
  caption="Figure 1. The nesting of conditions, experimental units, sessions, and trials comes from the study design. API calls, retries, and repeated requests are technical collection events; they do not automatically create independent experimental units."
  loading="lazy"
%}

APA's Journal Article Reporting Standards for quantitative research provide
checklists for experimental designs, random assignment, replication studies,
inclusion and exclusion criteria, and other information needed to evaluate a
study ([APA JARS–Quant](https://apastyle.apa.org/jars/quantitative)). These ideas
provide a methodological foundation for adapting the research record to
features specific to model inference.

## Order, counterbalancing, and practice effects

When the same experimental unit encounters several trials or conditions, what
happens earlier may affect what happens later. A response can differ because an
item appeared first rather than last, because one condition carried over into
the next, or because repeated exposure changed how the task was approached.
Counterbalancing varies the order systematically so that the condition of
interest is not always tied to the same position. Brooks (2012) discusses why
serial carryover makes condition order part of experimental design and how
different counterbalancing schemes address different orders of carryover
([Brooks, 2012](https://doi.org/10.1037/a0029310)).

In human research, _practice effects_ refer to changes associated with repeated
exposure to a test or task. Meta-analytic evidence shows that repeated testing
can change scores across a range of neuropsychological measures
([Calamia, Markon, & Tranel, 2012](https://pubmed.ncbi.nlm.nih.gov/22540222/)).
We should not assume that an LLM has the same learning or memory mechanism.
However, an analogous response pattern can arise when earlier instructions,
examples, answers, or feedback remain in its context. If every model call is
independent and contains no earlier history, that particular within-session
carryover route is absent; repetition can still reveal sampling variability or
item-specific sensitivity.

Counterbalancing is therefore not a decorative feature of the schedule. It is
one way to separate the effect of a condition from the effect of encountering
that condition at a particular point in a sequence. The exact order presented
to each unit must be saved, because a claim that order was randomized or
counterbalanced cannot be reconstructed from the final response alone.

## Replicability and generalizability answer different questions

Repeating one prompt asks whether a response recurs under another sample from
the same setup. Rerunning the same code and data asks whether a computational
result can be reproduced. A new study that collects new responses to address
the same scientific question asks about replicability. The National Academies
uses these distinctions while noting that fields sometimes use the terms
differently
([_Reproducibility and Replicability in Science_](https://www.nationalacademies.org/read/25303/chapter/3)).

Generalizability goes further: to what other models, model sizes, revisions,
prompts, tasks, stimuli, languages, backends, sampling settings, or deployment
contexts should the result apply?

For example, testing more items from one benchmark can show whether a pattern
extends beyond a small subset of that benchmark. It does not, by itself, show
that the result applies to other tasks, models, languages, or deployment
settings. Each of those broader claims requires variation in the corresponding
part of the design.

Yarkoni (2022) draws attention to a broader version of this problem:
researchers often make claims that extend beyond the particular stimuli,
tasks, participants, or sites included in a study without accounting for
variation across them
([Yarkoni, 2022](https://doi.org/10.1017/S0140525X20001685)). The same caution
applies when we move from one model, benchmark, or set of generation settings
to claims about LLM behavior more generally.

A precise result should therefore name the model, task, items, and conditions
to which the evidence directly applies. This is narrower than making a claim
about LLMs in general, but it is also a claim the experiment can support.

## Subjective and objective outcomes

An outcome can be scored using an objective rule when each item has a
prespecified answer and a deterministic procedure maps the model's response to
that answer. This avoids asking a new human or model judge to evaluate every
response. Even then, human judgment has not disappeared: researchers chose the
task, answer key, parser, invalid-response rule, and analysis. Those choices
should be specified before inspecting the desired result wherever possible.

Some outcomes cannot be reduced to a single answer key. Helpfulness, tone,
reasoning quality, similarity, or the presence of a theme may require human
judgment. This does not make them unusable. It means the construct, rubric,
coder training, blinding, disagreement procedure, and reliability of the
ratings become part of the measurement. Artstein and Poesio review agreement
measures and their assumptions for language annotation
([Artstein & Poesio, 2008](https://aclanthology.org/J08-4004/)).

Using another LLM as a judge does not automatically turn a subjective outcome
into an objective one. It replaces or supplements a human measurement process
with a model-based one that may itself be sensitive to prompt wording, answer
order, model version, and sampling settings. The judge, rubric, prompt,
position randomization, validation procedure, and raw judgments must therefore
be recorded.

## A successful request is not necessarily a valid observation

An HTTP success status means that the server handled the request. It does not
mean that the model followed the instructions, returned an analysable answer,
or remained within the intended experimental procedure. Technical success and
behavioral validity should be recorded separately.

The experiment must therefore define what counts as a valid response, how
unexpected outputs will be handled, and whether failed or retried requests are
included. These decisions should be made before the desired outcome is known
and applied consistently. The pipeline's
[`reproducibility`](https://github.com/marishkamehta/llm-blog-repo/tree/main/reproducibility/) templates
provide the run, exclusion, and deviation records needed to preserve these
decisions.

With both vocabularies in place, the next page returns to the practical
decision: where and how should the model run?

---

[← Previous: Infrastructure vocabulary](/blog/2026/infrastructure-vocabulary/) ·
[Series contents](/blog/2026/controlled-experiments/#what-this-series-covers) ·
[Next: Choosing where the model runs →](/blog/2026/choosing-a-backend/)
