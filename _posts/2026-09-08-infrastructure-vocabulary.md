---
layout: post
title: "The Infrastructure Terms I Needed at the Start"
date: 2026-09-08
categories: llm-behavior
description: "Practical infrastructure vocabulary for researchers beginning LLM experiments."
---

_Studying LLM Behavior · Vocabulary_

The difficulty I had at the beginning was not only that the infrastructure was
new to me. The guides I encountered used terms such as _model server_, _API_,
_endpoint_, and _context window_ before I knew why I needed any of them.

This page is for readers coming from behavioral and social science who may be
in the same position. It introduces the infrastructure terms needed for the
rest of the series without assuming a computer-science background. The next
page introduces the experimental terms that may be less familiar to technical
readers.

The definitions below are intentionally practical. Where a term has a more
specific disciplinary meaning, I link to a source and explain how I am using
it here.

{% include figure.liquid
  path="assets/img/blog/infrastructure-layers.svg"
  alt="A chat product and an experimental client both communicate through a request interface, which connects to a backend, a specific model revision, and computing hardware."
  caption="Figure 1. Names that are often used interchangeably refer to different layers. A reproducible experiment records the route from its client and request format through the backend, model revision, and compute environment."
  loading="lazy"
%}

## The model and the software around it

The first distinction is between the model we want to study and the
infrastructure that allows an experiment to interact with it.

| Term          | Meaning in this series                                                                                      |
| ------------- | ----------------------------------------------------------------------------------------------------------- |
| Model         | The learned system whose responses we want to study—for example, a GPT, Claude, Qwen, Llama, or Gemma model |
| Model weights | The numerical parameters learned during training and stored in files                                        |
| Model server  | Software that loads a model, receives a request, and returns the model's response                           |
| Backend       | Where and how the model is run, such as a direct hosted API, Ollama, Azure, or vLLM                         |
| Hosted model  | A model run on infrastructure operated by a provider and reached over a network                             |
| Local model   | A model run on a computer or server controlled by the researcher                                            |
| Container     | A packaged software environment containing an application and the dependencies it needs to run              |

Many people first encounter a model through a familiar chat product. ChatGPT
is a product through which people interact with OpenAI models; it is not the
name of one fixed underlying model. Claude.ai similarly provides a way to chat
with models in the Claude family. In everyday conversation, it is easy to use
the name of the product and the model interchangeably. In an experiment, we
need to separate them because the product can add instructions, tools,
interface features, and other behavior around the underlying model. OpenAI
maintains a separate [catalog of API models](https://developers.openai.com/api/docs/models),
while Anthropic distinguishes the
[Claude model family and Claude.ai chat interface](https://docs.anthropic.com/en/docs/welcome).

A model is also not the same thing as the software that runs it. Qwen is a
model family; Ollama and vLLM are tools that can run supported models; a direct
API provider makes selected models available without requiring the researcher
to deploy them; Azure is a cloud platform through which models can be deployed
and accessed. The same
model may be available through several of these routes.

Likewise, _local_ does not necessarily mean a laptop sitting on a desk. A model
running on a university GPU server may still be locally controlled for the
purposes of a study, even though the experiment reaches it over a network.
What matters is who operates the infrastructure and what control the
researcher has over it.

The [Docker overview](https://docs.docker.com/get-started/docker-overview/)
provides a fuller explanation of containers. Later in the series, the vLLM
setup uses a container to make its software environment easier to specify and
repeat.

## How the experiment communicates with the model

| Term             | Meaning in this series                                                                                         |
| ---------------- | -------------------------------------------------------------------------------------------------------------- |
| Request          | The instructions, conversation history, settings, and other information sent to a model server                 |
| Response         | What the server returns, including the generated text and technical metadata                                   |
| API              | The agreed structure through which software sends a request and receives a response                            |
| Endpoint         | The network address to which a particular kind of request is sent                                              |
| Client           | The small piece of experiment software that constructs requests and reads responses                            |
| Model registry   | A record connecting a short experiment name to the exact model, backend, endpoint, and credentials it requires |
| Message          | One structured contribution to a chat, usually containing a `role` and `content`                               |
| `system` role    | Instructions describing how the model should behave across the interaction                                     |
| `user` role      | Instructions or content presented as coming from the user side of the interaction                              |
| `assistant` role | A model response, or an earlier response included as part of the conversation history                          |
| Agent            | A larger system in which a model may be given instructions, tools, memory, and a procedure for taking actions  |

An API is an agreement between two pieces of software. The experiment sends
information in a defined structure, and the model server returns information
in another. The endpoint is the address where that exchange takes place; it is
not the model itself.

You may encounter the phrase _OpenAI-compatible API_. In this context,
_OpenAI-compatible_ describes the format of the exchange, not the organization
that developed the model. vLLM, for example, can serve non-OpenAI models
through interfaces modelled on several OpenAI API routes. Its documentation
also identifies which parts are supported and where behavior differs
([vLLM online-serving documentation](https://docs.vllm.ai/en/latest/serving/openai_compatible_server/)).

This pipeline is not restricted to OpenAI models. It can work with any model
and backend supported by the client. A common interface lets
the experimental task request a response without containing separate task
logic for every server.

### `system`, `user`, and `assistant` are message roles

A chat request is commonly represented as an ordered list of messages:

```json
[
  {
    "role": "system",
    "content": "Follow the experimental instructions exactly."
  },
  {
    "role": "user",
    "content": "Read the scenario and select one response."
  },
  {
    "role": "assistant",
    "content": "Response A"
  }
]
```

These labels describe positions in the chat format; they should not be read too
literally. A `user` message in an automated experiment may have been written by
the researcher and delivered by code rather than typed by a person. An
`assistant` message may be a response just generated by the model, or an older
model response inserted into the history before the next trial. A `system`
message usually contains instructions intended to shape the interaction as a
whole.

The labels also should not be taken to mean that the model has an intrinsic
understanding of itself as a user, assistant, participant, or agent. They are
structural cues supplied by the surrounding software. A `system` message can
tell the model what role or responsibility it is expected to take—for example,
to act as an experimental participant, follow a response format, or avoid using
outside information. The `assistant` marker indicates where a model response
belongs. These cues may influence the generated response, but they do not
establish that the model understands the assigned role as an identity or will
reliably follow it.

For the same reason, a model and an agent are not necessarily the same thing.
An agent is usually a larger system built around a model. It may add tools,
memory, external information, and a control loop that decides what happens
next. The model produces outputs within that system; it should not be assumed
to know the complete architecture or all of the processes operating around it.

The roles do not exist inside every model in exactly the same way. Chat models
ultimately process a sequence of tokens. A chat template converts the ordered
messages into the particular control tokens and formatting that a model was
trained to recognize. Some templates support a separate `system` role; others
may merge, reposition, or reject it. Additional roles may also exist on some
platforms. The
[Hugging Face chat-template documentation](https://huggingface.co/docs/transformers/chat_templating)
shows how common roles are converted into model-specific token sequences.

This has direct methodological consequences. Moving the same instruction from
`system` to `user`, changing the order of messages, or using a different chat
template can change the input the model actually receives. The complete ordered
message list, each role, and the applied template therefore belong in the
experimental record. A transcript containing only the visible prose may not be
enough to reconstruct the trial.

## What goes into a model response

| Term                  | Meaning in this series                                                                                                                |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Token                 | A unit into which model input and output are divided; it is not necessarily a complete word                                           |
| Context window        | The maximum amount of tokenized input and output that can fit in one model call                                                       |
| Chat template         | The model-specific formatting that turns roles and messages into the text or tokens presented to the model                            |
| Sampling              | The procedure used to select generated tokens from the model's possible continuations                                                 |
| Temperature           | A setting that reshapes the relative probabilities of possible next tokens before sampling                                            |
| `top_p`               | A setting that restricts sampling to the smallest group of likely next tokens whose cumulative probability reaches a chosen threshold |
| Maximum output tokens | A limit on how long the generated response may be                                                                                     |
| Seed                  | A value used to initialize a source of pseudorandomness                                                                               |

Tokens matter because instructions, previous messages, and responses must all
fit within the model's context window. A session that begins comfortably below
the limit can approach it as conversation history accumulates. Token counts
also affect the time and, for many hosted services, the cost of a run.

### Temperature and `top_p`

Temperature is sometimes described simply as a creativity setting. That
description is not precise enough for an experiment. At each step, a language
model assigns different probabilities to the tokens that could come next.
Temperature reshapes those relative probabilities. Lower temperatures
concentrate more probability on the options the model already ranks highly;
higher temperatures make lower-ranked options more competitive. The exact
behavior at zero and the permitted range can depend on the backend, so the
effective server configuration still needs to be checked rather than assumed.

`top_p`, sometimes called _nucleus sampling_, changes the set of tokens from
which a selection can be made. The possible next tokens are ordered from most
to least probable, and the server retains the smallest set whose cumulative
probability reaches the specified value. With `top_p = 0.9`, for example, the
long tail outside that probability mass is excluded before sampling. With
`top_p = 1.0`, this particular truncation does not remove any tokens.

Temperature and `top_p` therefore affect different parts of the same selection
process. They are not interchangeable, and changing both at once can make it
difficult to determine which setting produced a difference in behavior. A
study might hold one constant while manipulating the other, or deliberately
cross both in a factorial design. Either way, the decision belongs in the
experimental design and analysis plan—not only in a server configuration
file.

The values must also be recorded even when they are left at a default. Defaults
can come from the model, provider, client, or server and may not be the same
across backends. In the full NeuBAROCO demonstration in this series, for
example, temperature varied from 0.0 to 1.0 in increments of 0.1 while
`top_p` remained fixed at 1.0. That allowed the change in response stability to
be interpreted in relation to temperature without also changing the nucleus
used for sampling.

The current Hugging Face generation reference defines both parameters and
their configuration behavior
([Hugging Face generation documentation](https://huggingface.co/docs/transformers/main_classes/text_generation)).
`top_p` originates from work on nucleus sampling by Holtzman and colleagues
([Holtzman et al., 2020](https://arxiv.org/abs/1904.09751)).

{% include figure.liquid
  path="assets/img/blog/sampling-settings.svg"
  alt="Conceptual probability bars showing that temperature changes the relative spread of possible next tokens, whereas top-p removes the lower-probability tail from the candidate set."
  caption="Figure 2. Temperature reshapes relative token probabilities. Lower <code>top_p</code> values retain a smaller, more selective candidate set; higher values retain more of the probability distribution, and <code>top_p = 1.0</code> applies no nucleus truncation. The bars are conceptual and do not represent outputs from a particular model."
  loading="lazy"
%}

A seed can help control one source of computational variation, but recording a
seed does not make a run reproducible by itself. Model revision, server
version, hardware, prompt formatting, generation settings, concurrency, and
backend behavior may also matter.

The next vocabulary page explains the experimental language that the
infrastructure must preserve, including trials, sessions, conditions,
experimental units, repeated measures, and replication.

---

[← Previous: Why controlled experiments](/blog/) ·
[Series contents](/blog/2026/controlled-experiments/#what-this-series-covers) ·
[Next: Experimental vocabulary →](/blog/2026/experimental-vocabulary/)
