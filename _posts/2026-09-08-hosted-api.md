---
layout: post
title: "Connect Directly to a Hosted Model"
date: 2026-09-08
categories: llm-behavior
description: "How to connect the experimental pipeline directly to a hosted model."
---

_Studying LLM Behavior · Model setup_

For many readers, this will be the quickest way to run the pipeline with a
real model. No model needs to be downloaded and no server needs to be
maintained. The setup requires an account, an API key, an available model, and

This route is easier to set up than a local model, but the requests leave the
computer and the provider controls the environment in which the model runs.
That environment still needs to be considered and documented.

## Several services provide direct API access

This guide uses Google's Gemini API to demonstrate the direct hosted route.
Groq and OpenRouter can be connected through the same pipeline interface, but
they differ in the models they provide, how requests are processed, and what
happens to submitted data.

| Service           | Main distinction                                          |
| ----------------- | --------------------------------------------------------- |
| Google Gemini API | Direct access to Gemini models                            |
| Groq              | Hosted access to a changing selection of supported models |
| OpenRouter        | Access to models served by several underlying providers   |

Current information is available from:

- Google's documentation for its [OpenAI-compatible
  endpoint](https://ai.google.dev/gemini-api/docs/openai), [rate
  limits](https://ai.google.dev/gemini-api/docs/rate-limits), and [pricing and
  data use](https://ai.google.dev/gemini-api/docs/pricing);
- Groq's [compatibility guide](https://console.groq.com/docs/openai), [model
  catalog](https://console.groq.com/docs/models), and [rate-limit
  table](https://console.groq.com/docs/rate-limits); and
- OpenRouter's [client configuration](https://openrouter.ai/docs/quickstart),
  [free-model limits](https://openrouter.ai/docs/faq), and [provider data
  policies](https://openrouter.ai/docs/guides/privacy/provider-logging/).

Model availability, limits, prices, and data-use terms can change. Check the
selected provider's documentation when the study is prepared and again before
data collection begins.

## Decide what can be sent before creating the connection

A direct hosted request sends the instructions, stimuli, session history, and
any other material in the prompt to an external service. Identifiable,
sensitive, restricted, or participant-derived information should not be sent
unless that use is covered by the study's consent materials, ethics approval,
institutional policy, and the provider's current terms.

Free access does not necessarily carry the same limits or data-use terms as
paid access. Price, privacy, and suitability for the study should be considered
separately.

## Add Gemini to the model registry

Create an API key through the provider's account console. The key should not be
placed in `model-registry.yaml`, a notebook, a script, a screenshot, or version
control. The registry stores only the name of the environment variable from
which the pipeline will read it.

Add the Gemini entry below, replacing `REPLACE_WITH_CURRENT_GEMINI_MODEL` with
the name of the model that will be used. Confirm the current model name in
Google's documentation before continuing.

```yaml
gemini-hosted:
  backend: openai-compatible
  base_url: https://generativelanguage.googleapis.com/v1beta/openai
  served_name: REPLACE_WITH_CURRENT_GEMINI_MODEL
  key_env: GEMINI_API_KEY
```

The same registry structure can be used with another compatible provider by
changing the base URL, model name, and environment-variable name. Keep only the
provider used by the experiment in the working registry.

## Make the key available for the current terminal session

For example, if the Gemini entry is being used on macOS or Linux:

```bash
read -s GEMINI_API_KEY
export GEMINI_API_KEY
```

The same Gemini example in Windows PowerShell is:

```powershell
$secureKey = Read-Host "API key" -AsSecureString
$env:GEMINI_API_KEY = [System.Net.NetworkCredential]::new("", $secureKey).Password
```

The variable lasts for the current terminal session. There is no need to print
the key to confirm that it was set.

## Send one non-study request

Return to the terminal where the
[`llm-behavior-pipeline`](../../llm-behavior-pipeline/) environment is active.
The smoke test below sends “Say hello in three words.” It contains no study
material, but the provider may count or bill the request.

Replace `gemini-hosted` with the registry name selected above:

```bash
python smokes/smoke_chat.py --model gemini-hosted
```

A response confirms that the pipeline can use the key, reach the endpoint, and
request the selected model. It does not confirm that the provider supports
every setting required by the experiment. Test the complete study
configuration before collecting data.

Temperature 0 does not guarantee identical responses from a hosted service.
The provider may update a model, change the serving system, or route a request
differently. Repeated trials and a dated record of the environment remain
necessary.

## Preserve the hosted environment with the responses

The study record should preserve:

- the provider, endpoint, model name, and collection date;
- generation and safety settings;
- the service tier, rate limits, and available version information;
- request IDs, errors, token use, and returned metadata; and
- the applicable data-governance decision.

Never include the API key in the study record.

OpenRouter requires an additional decision because it can route a model through
more than one provider. Automatic routes, including a route that selects any
currently free model, are useful for exploration but do not hold the model
constant for a controlled experiment. Use a specific model and preserve any
returned provider information when the study requires that control.

## Ask an agent to connect one direct provider

An agent can check the configuration and run the non-study smoke test without
being given the credential itself.

```text
I have downloaded and tested llm-behavior-pipeline with its mock backend. I
want to connect it to [PROVIDER] through that provider's direct hosted API.

Use only the provider's current official documentation. Confirm the exact API
base URL, an available model identifier, supported generation settings,
current rate limits, price or free-tier conditions, and data-use terms. Tell me
which points I need to resolve before sending study material.

Add one sanitized entry to model-registry.yaml. The entry must read the API key
from an environment variable. Do not ask me to paste the key into chat or
write it into a file, notebook, command history, or version control.

After I have set the environment variable and explicitly authorized one
request, run only the pipeline's non-study smoke test. Report the model name,
endpoint, HTTP result, returned model and provider metadata, request ID, rate
limit headers, token usage, and any unsupported settings. Do not send study
material, create a paid resource, change an account setting, or make additional
requests without my approval.
```

A successful smoke test confirms that the pipeline can reach the model and save
its response. A behavioral experiment must also vary a condition, repeat the
procedure, and connect each response to the trial that produced it.

---

[← Previous: Testing the pipeline](/blog/2026/mock-pipeline/) ·
[Series contents](01-controlled-experiments.md#what-this-series-covers) ·
**Next:** [Position bias](/blog/2026/position-bias/) or
[belief bias](/blog/2026/belief-bias/) →
