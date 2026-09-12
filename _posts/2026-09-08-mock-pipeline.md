---
layout: post
title: "Download and Test the Pipeline Without a Model"
date: 2026-09-08
categories: llm-behavior
description: "How to validate an experimental pipeline before connecting a live model."
---

_Studying LLM Behavior · Pipeline setup_

Before connecting the pipeline to a local model, hosted service, or inference
server, run it without a model. This checks that the instructions, trials,
session history, responses, and research records move through the pipeline as intended.

The template does this through a **mock backend**. It follows the same route
through the pipeline as a real model, but does not load a model or contact an
external service. Every request receives the same response: `MOCK_RESPONSE`.
At this stage, we are not collecting behavioral data. We are checking the
experimental procedure before adding model behavior to it.

{% include figure.liquid
  path="assets/img/blog/mock-pipeline.svg"
  alt="Experimental materials pass through the pipeline to a mock backend and saved record. The mock can test task order, history, request construction, and recording, but not model understanding, answer validity, behavioral effects, or backend-specific behavior."
  caption="Figure 1. The mock backend tests whether the experimental procedure works as intended. It does not provide evidence about how a model behaves."
  loading="lazy"
%}

## A mock separates procedural errors from model behavior

Suppose an expected trial is missing from a saved session. With a real model
attached, I might initially wonder whether the request exceeded a context
limit, whether the server rejected it, or whether the response parser failed.
If the same problem appears with the mock, none of those explanations can be
responsible. The error is in the task or recording procedure.

This first pass can check whether:

- the study creates the expected sessions and trials;
- conditions and stimuli are assigned in the intended order;
- instructions and previous turns appear in the correct model input;
- each response is attached to the correct trial;
- request and response records are written to the intended location; and
- an interrupted run can be identified without silently losing observations.

It cannot show whether a real model understands the instructions, produces a
valid answer, or behaves consistently. It also cannot reveal service-specific
limits or differences in prompt formatting. Those checks begin after a real
backend is connected.

## Download the pipeline template

Start with the companion repository,
[`llm-blog-repo`](https://github.com/marishkamehta/llm-blog-repo) on GitHub. Select **Code > Download ZIP** to download the
source code as a ZIP file and unzip it somewhere you can easily find again.

Before going further, make sure that Python 3.10 or later is installed. If it is
not, download it from the official [Python website](https://www.python.org/downloads/).

We now need to run a few commands from inside the downloaded pipeline folder.
This is the only reason for opening a terminal at this point: it tells the
computer which folder we are working in. In Windows File Explorer, right-click
inside the folder and select **Open in Terminal**. On macOS, use **New Terminal
at Folder** from Finder Services. If a terminal is already open, `cd` can be
used to move into the folder.

To check that the terminal opened in the right place, run `pwd` on macOS or
Linux, or `Get-Location` in PowerShell. The path it returns should end with the
name of the downloaded pipeline folder.

## Install the template in an isolated environment

On macOS or Linux, run:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -e '.[test]'
```

On Windows PowerShell, run:

```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -e ".[test]"
```

The `.venv` directory keeps this project's packages separate from other Python
projects on the computer. Activating it also makes `python` refer to the Python
installation inside that directory.

## Send one request without compute, credentials, or cost

With the environment active, run the smallest available check:

```bash
python smokes/smoke_chat.py --model mock
```

The result should look like this:

```text
model: mock
reply: 'MOCK_RESPONSE'
metadata: {'model': 'mock', 'usage': None}
```

No model has been downloaded, no API key has been supplied, and no study
material has left the computer. This output only confirms that Python can load
the pipeline, select the mock backend, send it a request, and receive a record
in the expected form.

The word _smoke_ here comes from software testing. A smoke test is a small,
early check that the main path through a system works. It does not establish
that every part of the system is correct.

## How session history is carried forward

Some behavioral tasks require each trial to build on what happened earlier in
the session. The template represents this history as a **trajectory**: an
ordered list containing the instructions, trial prompts, and model responses
seen so far.

In this pipeline, we do not rely on the model server to remember the
participant or retrieve an earlier call. The experiment carries the session
history forward by adding the new trial to the trajectory and sending the full
trajectory to the model. The response is then added to the same trajectory,
and that updated history is sent with the next trial.

This also gives the researcher direct control over what the model can use from
earlier trials. A new trajectory should be created when a new independent
session begins. Otherwise, information from one session could inadvertently
carry into the next.

This differs from sending each trial as an independent prompt, where the model
receives no information from earlier trials. It also differs from relying on a
service to store the conversation behind a conversation ID. Keeping the
trajectory in the experiment makes the exact history visible in the research
record and allows the same session logic to be used across different backends.

There can also be a computational advantage, although it depends on the
backend. Because each new request begins with the same unchanged history, a
cache-aware server can reuse work it has already done on that shared beginning.
vLLM calls this **automatic prefix caching**. Its documentation identifies
multi-round conversations as one use case because the earlier history does not
have to be processed again from the beginning on every turn
([vLLM documentation](https://docs.vllm.ai/en/latest/features/automatic_prefix_caching/)).
This reduces the work needed to process the input, but it does not reduce the
time needed to generate new response tokens. It also offers little benefit if
the earlier part of the trajectory keeps changing.

The main reason for carrying the trajectory explicitly is experimental control
and reproducibility. Faster processing is an additional benefit when the
backend supports this form of caching.

The template includes a minimal example:

```bash
python examples/growing_trajectory.py
```

It produces two mock responses. More importantly, the code shows where the
history is assembled:

```python
trajectory = []

for trial_text in ["This is trial one.", "This is trial two."]:
    trajectory.append({"role": "user", "content": trial_text})
    result = model.respond_with_metadata(trajectory, config=config)
    trajectory.append({"role": "assistant", "content": result.text})
```

After the first response, both the first trial and its response remain in
`trajectory`. The second request therefore contains that history as well as the
new trial. The pipeline preserves the trajectory it is given, but the decision
about what belongs in that history remains part of the experimental design.

## Run the automated checks before changing the backend

Run the test suite from the
[`llm-blog-repo`](https://github.com/marishkamehta/llm-blog-repo) folder:

```bash
python -m pytest
```

The tests check two details that are easy to overlook: the mock must work
without a model registry, and credentials must be removed from the request
metadata before it is saved. A passing test suite does not validate a new
experiment, but it establishes that these shared pieces of the template still
work before task-specific code is added.

The mock remains useful after a real model is connected. It provides a fast way
to test changes to trial logic, randomization, file handling, and session
construction without spending compute or making paid calls. The same task can
then be pointed to a local runner, hosted endpoint, or inference server through
the model registry.

## Ask an agent to complete the offline setup

After downloading and extracting the pipeline, an agent can carry out the
remaining offline setup and run the checks below. The prompt keeps this stage
offline, asks the agent to report any changes, and authorizes installation only
inside the pipeline's virtual environment.

```text
I am working in the llm-blog-repo repository. Inspect its README
before making changes.

Set up a Python virtual environment inside the pipeline directory, install
the template and its test dependencies into that environment, and run:

1. the mock smoke test;
2. the growing-trajectory example; and
3. the complete test suite.

Do not connect to a model service, download model weights, request
credentials, make an API call, or modify my experimental materials. If
package installation requires internet access, tell me before proceeding.
Report the commands used, the output of each check, and any files created
or changed. If something fails, diagnose the failure but do not conceal it
or weaken a test merely to make it pass.
```

Once these checks pass, the predictable mock response can be replaced with a
response from a real model without changing the task logic.

## Citation

If you use the pipeline, cite the software using the metadata in the
repository's [CITATION.cff](https://github.com/marishkamehta/llm-blog-repo/blob/main/CITATION.cff):

```text
Dennis, D. K., & Mehta, M. M. (2026). LLM Behavioral Pipeline Template
(Version 0.1.0) [Computer software]. GitHub.
https://github.com/marishkamehta/llm-blog-repo
```

Record the Git commit hash used for your study alongside this citation so that
readers can identify the exact code version.

---

[← Previous: Choosing where the model runs](/blog/2026/choosing-a-backend/) ·
[Series contents](/blog/2026/controlled-experiments/#what-this-series-covers)

**Choose a model setup:** [Hosted API](/blog/2026/hosted-api/) ·
[Ollama](/blog/2026/ollama/) · [Azure](/blog/2026/azure/) · [vLLM](/blog/2026/vllm/)
