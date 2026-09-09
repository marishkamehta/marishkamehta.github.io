---
layout: post
title: "Where Should the Model Run?"
date: 2026-09-08
categories: llm-behavior
description: "How to choose where an LLM runs for a behavioral experiment."
---

_Studying LLM Behavior · Choosing infrastructure_

Several factors shape where an LLM experiment can run. These include the
computing power available, cost, privacy, the number and structure of the model
calls, and the degree of control required by the study.

A model can run on the computer in front of me, on a server I control, or on
infrastructure managed by someone else. Those routes are not interchangeable.
They affect how much the study costs, how quickly it can run, what information
leaves the local environment, and how much of the model environment I can
inspect and preserve.

## Translate the experimental design into infrastructure requirements

The first step is to work out the experimental design and the resources needed
to support it. This means estimating how many sessions and trials the study
will contain, how many model calls they will require, and whether those calls
are independent or share a growing history. It also means considering how long
the instructions and responses may become. A short pilot with independent
prompts is not the same infrastructure problem as hundreds of sessions that
grow after every trial.

The design then has to be considered alongside the available resources.
Several sessions may need to run at the same time. Prompts may contain material
that cannot be sent to an external provider. The study may have little or no
computing budget, or it may already have access to an institutionally supported
cloud service. It may also require settings that a particular service does not
expose.

These questions narrow the options more usefully than asking whether a machine
has a GPU. A smaller local model may be enough for one study. Another may need
hosted compute because the model will not fit locally. A third may need a
dedicated server because the number and structure of the requests make serving
efficiency part of what determines whether the experiment is feasible.

## The routes differ in both setup and control

The series covers five routes: no-model testing, direct access to a hosted API,
an accessible local runner, an institutionally managed cloud deployment, and a
configurable inference server. These are categories. The named services are
the particular examples used to make each route concrete.

{% include figure.liquid
  path="assets/img/blog/backend-routes.svg"
  alt="A behavioral task sends requests through a shared client and model registry to a mock backend, direct hosted API, local runner, managed cloud deployment, or inference server; each route feeds a common research record."
  caption="Figure 1. The experimental task and research record remain stable while the route used to obtain a response changes. The named services are examples, not the only options in their categories."
  loading="lazy"
%}

The amount of setup is only one part of this decision. The simplest route to
start may provide less control over the environment in which the model runs.

| Route                             | Initial setup                                            | Local computing requirement                      | Control and privacy                                                                    |
| --------------------------------- | -------------------------------------------------------- | ------------------------------------------------ | -------------------------------------------------------------------------------------- |
| Direct hosted API                 | Usually the easiest real-model route                     | Minimal                                          | Requests leave the computer, and the provider controls much of the serving environment |
| Local runner                      | Install the runner and download a model                  | Enough memory and storage for the selected model | The model runs locally, providing greater privacy and local control                    |
| Existing managed cloud deployment | Straightforward once access has been granted             | Minimal                                          | The institution may provide governance and deployment records                          |
| New managed cloud deployment      | Requires cloud resources, permissions, and configuration | Minimal on the researcher's computer             | More deployment control, but substantially more administrative setup                   |
| Configurable inference server     | The most technical route                                 | Suitable local or remote compute                 | The researcher controls most serving decisions                                         |

### Use a mock backend to verify the procedure

The mock backend does not load or contact a model. It returns responses that
are deliberately predictable. That may sound too simple to be useful, but it
lets me answer a necessary question first: does the experiment work?

I can check that trials appear in the intended order, conditions are assigned
correctly, session history is preserved, and outputs are written where I
expect. If something fails, I can investigate the task without also wondering
whether the problem came from a model download, a network request, a credential,
or GPU memory. The mock cannot produce behavioral data. Its purpose is to make
sure the procedure is ready before behavioral data are collected.

### Connect directly to a hosted model for the simplest setup

For many readers, a direct hosted API will be the quickest way to obtain a
response from a real model. There is no model to download and no server to
operate. The researcher creates an account, obtains an API key, selects an
available model, and adds the provider's endpoint to the pipeline.

In this series, we present Google's Gemini API as the worked example. Groq and
OpenRouter offer other ways to reach hosted models without running one locally.
The practical details differ across services, including which models are
available, which settings can be controlled, and what happens to submitted
data. These details can also change. Model access and service terms should therefore be
checked when a study is prepared, rather than inferred from an older guide.

This route is easy to start because the provider operates the infrastructure.
That also means the provider controls much of the environment in which the
model runs. Study material leaves the local computer, and a model or serving
system may be updated without the researcher controlling that change. Direct
hosted access is therefore technically accessible, but it still requires a
data-governance and reproducibility review before collection begins.

### Run the model locally when local control matters

The local route replaces the mock with a real model while keeping everything
else small. The walkthrough uses Ollama, which handles much of the
work involved in downloading and running a supported model, works across
macOS, Windows, and Linux, and exposes an API that the experiment can call. The
official [Ollama quickstart](https://docs.ollama.com/quickstart) and
[API introduction](https://docs.ollama.com/api/introduction) describe the
current setup.

This route is useful when prompts need to remain on the computer, internet
access is unreliable, or greater control over the model environment is needed.
It takes more work to start than a hosted API because the runner must be
installed, the model downloaded, and the available memory and storage checked.
Before a study begins, it is also important to confirm that the model can
handle the length of the task, respond quickly enough, and expose the settings
that need to be controlled and recorded.

The local setup in this guide uses Ollama. For readers who prefer a graphical
interface, LM Studio also provides a local API server
([LM Studio documentation](https://lmstudio.ai/docs/developer/rest)).
`llama.cpp` offers a lower-level alternative for compatible models.

### Connect to a managed cloud deployment when one is available

An institutionally managed deployment also runs the model elsewhere, but it is
not the same as opening an individual account with a direct API provider. In
this series, Azure is the worked example.

If an approved deployment already exists, connecting the pipeline may be
straightforward. Creating a new deployment is not. It can require a cloud
subscription, resource and model deployment, permissions, authentication,
quota, content-filter configuration, billing arrangements, and institutional
approval. This route is most accessible when a university or organization has
already established those resources and their data-governance procedures.

An Azure deployment contains more than a model name. It can also specify a
model version, capacity arrangement, filtering configuration, and rate limits.
Microsoft describes these components in its
[Foundry endpoint documentation](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/endpoints).
This is why reporting that a study used “Azure” or a particular model is not
enough to reconstruct the environment.

Azure is one managed cloud route, not the only one. Other possibilities include
[Amazon Bedrock](https://docs.aws.amazon.com/bedrock/),
[Google Vertex AI](https://docs.cloud.google.com/sdk/gcloud/reference/ai/model-garden/models/deploy),
[Hugging Face Inference Endpoints](https://huggingface.co/docs/inference-endpoints/about),
and dedicated deployments offered by other cloud platforms. A direct provider
API and a managed deployment do not offer the same setup or degree of control.
If an institution already supports one of these services, that may be a better
starting point than adopting Azure solely to follow this example.

### Use a configurable server for larger or repeated workloads

For larger or more demanding local workloads, the question is no longer only
whether the model can run. It is also whether the server can handle the way the
experiment uses it. The demonstrations in this project were run through vLLM,
which is why it is the worked example for a configurable inference server.

vLLM exposes decisions about context length, GPU-memory use, parallelism, and
caching. Its prefix caching can avoid recomputing an unchanged beginning shared
by several requests, which is potentially useful when the same instructions or
session history recur. The official documentation covers
[online serving](https://docs.vllm.ai/en/latest/serving/openai_compatible_server/)
and [automatic prefix caching](https://docs.vllm.ai/en/latest/features/automatic_prefix_caching/).

The additional control comes with additional work. The model must be supported,
fit the available hardware, and be launched with settings that can later be
reconstructed. vLLM can run on a local GPU, but it can also run on a remote
server. It is therefore not simply the “has a GPU” option. A simpler local
runner is usually easier for learning or piloting. A server such as vLLM becomes
useful when the number, concurrency, or repeated structure of the requests
makes those serving controls important.

vLLM is not the only server in this category. SGLang and Hugging Face Text
Generation Inference are alternatives, and `llama.cpp` can also expose a
server. The categories overlap because a managed hosting platform may itself
run one of these engines behind the endpoint visible to the researcher.

## Keep the experimental task stable across routes

Changing the route should not require rewriting the experiment. The task sends
requests through a shared client, while the model registry records where those
requests should go. This lets the same trial logic work with a mock, local
runner, hosted service, or inference server.

The shared interface does not make the routes scientifically equivalent. They
may differ in defaults, prompt formatting, filtering, model revisions, and
concurrency behavior. Those differences still have to be recorded. The point
is to keep them in the model environment rather than allowing them to become
untracked changes to the experimental procedure.

## Prompt an agent to assess the requirements

An agent can help inspect the available computer and compare possible routes.
The following prompt asks only for inspection and advice; it does not authorize
installation, uploads, paid calls, or changes to the computer.

```text
I want to run language models as participants in a behavioral experiment, but
I am new to model infrastructure.

Inspect my computer and the project files I identify without changing
anything. Compare these possible starting routes:

- no-model testing through a mock backend
- direct access to a hosted API, such as the Gemini API, Groq, or OpenRouter
- an accessible local runner, such as Ollama or LM Studio
- an existing managed cloud deployment, such as Azure
- a configurable local or remote inference server, such as vLLM

Consider my operating system, CPU, RAM, GPU and GPU memory, free disk space,
expected context length, number of model calls, desired speed, budget, privacy
requirements, and reproducibility requirements.

Recommend one starting route and one fallback. State your assumptions, explain
unfamiliar terms in plain language, and report any commands used for
inspection. If another locally available or institutionally supported route
would fit better, identify it rather than forcing the choice into these four
examples. Do not install software, alter files, expose credentials, send study
data to an external service, or incur costs.
```

The next page tests the pipeline with a mock before any local or hosted model
is involved.

---

[← Previous: Experimental vocabulary](/blog/2026/experimental-vocabulary/) ·
[Series contents](01-controlled-experiments.md#what-this-series-covers) ·
[Next: Testing the pipeline →](/blog/2026/mock-pipeline/)
