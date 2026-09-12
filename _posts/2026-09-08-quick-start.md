---
layout: post
title: "Quick Start Guide I: Hello LLM"
date: 2026-09-08
categories: llm-behavior
description: "A practical first step for interacting with a hosted language model from Python."
---

You can start interacting programmatically with an LLM using only a few lines
of code and a model hosted on Google's free tier. This guide shows you how to:

1. obtain an API key from Google AI Studio; and
2. use Python to send a message to the model.

## 1. Get a free key from Google AI Studio

You will use an **API endpoint** and an **API key** to interact with the model.
An API endpoint is the web address that receives your requests. An API key
authenticates those requests, much like a password.

To obtain a key:

1. Go to [Google AI Studio](https://aistudio.google.com/) and sign in.
2. Select **Get API Key**, then **Create API key**, and copy the new key.
3. Make sure you are using the free tier. Do not enable billing for this quick
   start. The free tier already has usage limits, which act as a ceiling so a
   mistake in your code cannot run up a bill.

Treat the key like a password. Do not paste it into your code, a notebook, a
screenshot, or version control. In the next step you place it in an environment
variable instead, so it stays out of the code you write and share. You can
delete and recreate the key if it is compromised.

## 2. Send a message to the LLM

Install the official Gemini library:

```bash
pip install google-genai
```

Make your key available to the current terminal session, without printing it or
writing it into a file:

```bash
# macOS or Linux. Paste the key when prompted, then press Enter.
read -s GEMINI_API_KEY
export GEMINI_API_KEY
```

Start a Python interpreter, then paste the following code:

```python
import os
from google import genai

client = genai.Client(api_key=os.environ["GEMINI_API_KEY"])
response = client.models.generate_content(
    model="gemini-3.6-flash",
    contents="Hello LLM",
)
print(response.text)
```

Run it, and the model's reply prints to your screen.

### Notes

- `google-genai` is Google's official Python library for the Gemini API. We use
  it here for simplicity, but the rest of the series uses a common interface
  that can connect to several providers.
- API limits can include requests per minute (RPM), input tokens per minute
  (TPM), and requests per day (RPD). One call to
  `client.models.generate_content` is one request; the size of its input affects
  token usage. Check the [current limits in Google AI
  Studio](https://aistudio.google.com/rate-limit?timeRange=last-28-days) before
  running repeated requests. Limits vary by model, project, and billing tier.

---

[Series contents](/blog/2026/controlled-experiments/#what-this-series-covers) ·
[Next: A first experiment with Silico →](/blog/2026/first-experiment/)
