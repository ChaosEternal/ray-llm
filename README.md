# Aviary - Study stochastic parrots in the wild

Go on bird watch right now: [🦜🔍 Aviary 🦜🔍](http://aviary.anyscale.com/)

Aviary is an app that lets you interact and deploy large language models (LLMs) in a single place. 
You can compare the outputs of different models directly, rank them by quality,
get a cost and latency estimate, and more. In particular, it offers good support for 
Transformer models hosted on [Hugging Face](http://hf.co) and in many cases also 
supports [DeepSpeed](https://www.deepspeed.ai/) inference acceleration. 

Aviary is built on top of [Ray](https://ray.io) by [Anyscale](https://anyscale.com). This template uses the `anyscale/aviary:latest-tgi` docker image.

## Table of Contents

* [Aviary User Guides](#Aviary-User-Guides)
	* [Deploy Aviary ](#Deploy-Aviary)
		* [Query Aviary](#Query-Aviary)
  	* [Deploy on Anyscale Service](#Deploying-on-Anyscale-Service)
* [Aviary Reference](#Aviary-Reference)
	* [Installing Aviary](#Installing-Aviary)
	* [Running Aviary Frontend locally](#Running-Aviary-Frontend-locally)
	* [Aviary Model Registry](#Aviary-Model-Registry)
   	* [CLI examples](#CLI-examples)
	* [Usage stats collection](#Usage-stats-collection)
	* [Getting Help and Filing Bugs / Feature Requests](#Getting-Help-and-Filing-Bugs-/-Feature-Requests)

# Aviary User Guides

For a video introduction, see the following intro. Note: There have been some minor changes since the video was recorded. The guide below is more up to date. 

[![Watch the video](https://img.youtube.com/vi/WmqPfQOXJ-4/0.jpg)](https://www.youtube.com/watch?v=WmqPfQOXJ-4)

## Deploy Aviary 
From the terminal use the aviary CLI to deploy a model:

```shell
# Deploy the LightGPT model. 
aviary run --model ./models/static_batching/amazon--LightGPT.yaml
```

You can deploy any model in the `models` directory of this repo, 
or define your own model YAML file and run that instead.

### Query Aviary

From the head node, run the following commands. 

```shell
export AVIARY_URL="http://localhost:8000"

# List the available models
aviary models
amazon/LightGPT

# Query the model
aviary query --model amazon/LightGPT --prompt "How do I make fried rice?"
```
```text
amazon/LightGPT:
To make fried rice, start by heating up some oil in a large pan over medium-high
heat. Once the oil is hot, add your desired amount of vegetables and/or meat to the
pan. Cook until they are lightly browned, stirring occasionally. Add any other
desired ingredients such as eggs, cheese, or sauce to the pan. Finally, stir
everything together and cook for another few minutes until all the ingredients are
cooked through. Serve with your favorite sides and enjoy!
```

You can also use `aviary query` with certain LangChain-compatible APIs.
Currently, we support the following APIs:
* openai (`langchain.llms.OpenAIChat`)

```shell
# langchain is an optional dependency
pip install langchain

export OPENAI_API_KEY=...

# Query an Aviary model and OpenAI model
# [PROVIDER]://[MODEL_NAME]
aviary query --model amazon/LightGPT --model openai://gpt-3.5-turbo --prompt "How do I make fried rice?"
```
# Deploying on Anyscale Service

To deploy an application with one model on an Anyscale Service you can run:

```shell
anyscale service rollout -f template/service.yaml --name {ENTER_NAME_FOR_SERVICE_HERE}
```

This is setup to run the amazon/LightGPT model, but can be easily modified to run any of the other models in this repo.
In order to query the endpoint, you can modify the `template/request.py` script, replacing the query url with the Service URL found in the Service UI.

# Aviary Reference

## Installing Aviary

To install Aviary and its dependencies, run the following command:

```shell
pip install "aviary @ git+https://github.com/ray-project/aviary.git"
```

The default Aviary installation only includes the Aviary API client.

Aviary consists of a backend and a frontend, both of which come with additional
dependencies. To install the dependencies for both frontend and backend for local
development, run the following commands:

```shell
pip install "aviary[frontend,backend] @ git+https://github.com/ray-project/aviary.git"
```

The backend dependencies are heavy weight, and quite large. We only recommend installing
them on a cluster.

## Aviary Model Registry

Aviary allows you to easily add new models by adding a single configuration file.
To learn more about how to customize or add new models, 
see the [Aviary Model Registry](models/README.md).

## CLI examples


#### Running two models on the same prompt

```shell
aviary query --model mosaicml/mpt-7b-instruct --model RWKV/rwkv-raven-14b \
  --prompt "what is love?"
```
```text
mosaicml/mpt-7b-instruct:
love can be defined as feeling of affection, attraction or ...
RWKV/rwkv-raven-14b:
Love is a feeling of strong affection and care for someone or something...
```

#### Running a batch-query of two prompts on the same model

```shell
aviary query --model mosaicml/mpt-7b-instruct \
  --prompt "what is love?" --prompt "why are we here?"
```

#### Running a query on a text file of prompts

```shell
aviary query --model mosaicml/mpt-7b-instruct --prompt-file prompts.txt
```

#### Evaluating the quality of responses with GPT-4 for evaluation

```shell
 aviary evaluate --input-file aviary-output.json --evaluator gpt-4
```

This will result in a leaderboard-like ranking of responses, but also save the
results to file:

```shell
What is the best indie band of the 90s?
                                              Evaluation results (higher ranks are better)                                               
┏━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Model                    ┃ Rank ┃                                                                                            Response ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ mosaicml/mpt-7b-instruct │ 1    │  The Shins are often considered to be one of the greatest bands from this era, with their album 'Oh │
│                          │      │        Inverted World' being widely regarded as one of the most influential albums in recent memory │
│ RWKV/rwkv-raven-14b      │ 2    │ It's subjective and depends on personal taste. Some people might argue that Nirvana or The Smashing │
│                          │      │                       Pumpkins were the best, while others might prefer Sonic Youth or Dinosaur Jr. │
└──────────────────────────┴──────┴─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

You can also use the Gradio API directly, by following the instructions
provided in the [Aviary documentation](https://aviary.anyscale.com/?view=api).

## Usage stats collection

Aviary backend collects basic, non-identifiable usage statistics to help us improve the project.
The mechanism for collection is the same as in Ray.
For more information on what is collected and how to opt-out, see the
[Usage Stats Collection](https://docs.ray.io/en/latest/cluster/usage-stats.html) page in
Ray documentation.

## Getting Help and Filing Bugs / Feature Requests

We are eager to help you get started with Aviary. You can get help on: 

- Via Slack -- fill in [this form](https://docs.google.com/forms/d/e/1FAIpQLSfAcoiLCHOguOm8e7Jnn-JJdZaCxPGjgVCvFijHB5PLaQLeig/viewform) to sign up. 
- Via [Discuss](https://discuss.ray.io/c/llms-generative-ai/27). 

For bugs or for feature requests, please submit them [here](https://github.com/ray-project/aviary/issues/new).

We have people in both US and European time zones who will help answer your questions. 

