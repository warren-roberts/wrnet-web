+++
date = '2026-08-13'
draft = false
title = 'Building a Local Hermes Agent'
summary = 'A work-in-progress agent stack built around local inference, local tools, and explicit control over where requests go.'
+++

![An abstract local AI agent connects to search, extraction, browser, and data-service tools in a home-lab environment.](local-hermes-agent.png)

## Local means more than the model

I have been working on a local Hermes agent setup built around a simple idea: **running the model locally is only half the job**.

An agent can run inference on hardware sitting ten feet away and still quietly hand search, browsing, extraction, or automation work to hosted services. If the goal is a system I can understand, control, and eventually rely on, the tool paths matter just as much as the inference path.

My setup deliberately separates the **agent runtime** from the **LLM hardware**.

Hermes runs on a dedicated Ubuntu machine. It does not host the primary model itself. Instead, Hermes acts as the orchestration layer: it maintains the working context, decides when tools are useful, calls MCP services, controls browser activity, and sends inference requests across my local network.

The primary model currently runs on a separate Windows 11 workstation:

* **Ubuntu machine:** Hermes Agent and local tool orchestration
* **Windows 11 workstation:** Gemma 4 31B running under Ollama
* **MacBook Pro:** planned additional Qwen models for specialized workloads
* **Cloud fallback:** Claude Sonnet when the local path is unavailable or unsuitable

Gemma is exposed by Ollama over an OpenAI-compatible endpoint, so from Hermes' perspective the model is a network service. That separation is important. The model reasons; Hermes gives it a disciplined way to act.

It also means I can change the model infrastructure without rebuilding the agent infrastructure around it.

## Four paths, four jobs

The interesting part is the surrounding tool belt.

Search goes through a self-hosted **SearXNG** instance. Its job is discovery: find current results, URLs, and leads without turning every search into another hosted API request.

Extraction is a separate concern. For that, I am using **Scrapling inside a Docker-backed MCP server**. Hermes can launch it as a standard input/output MCP process and use it when a search result needs to become usable page content.

Keeping that capability containerized has a few advantages. Browser and scraping dependencies stay isolated, the host remains cleaner, and extraction remains a distinct service rather than becoming another pile of dependencies inside Hermes itself.

Then there is the browser path.

**Local Chromium** handles interactive cases: JavaScript-heavy sites, navigation, authentication flows, or anything that actually requires a browser session.

Those are intentionally separate responsibilities:

**SearXNG discovers. Scrapling extracts. Chromium interacts.**

Treating them as interchangeable "web tools" makes it much harder to understand what an agent is actually doing.

I have also been adding narrower MCP services for specific jobs, including a local job-search service backed by its own API and database. That is another architectural pattern I want to keep: general-purpose tools for the web, focused tools for workflows I expect to repeat.

## The architecture is the feature

At a high level, the stack looks like this:

```text
                         Local network
                              │
                    ┌─────────▼─────────┐
                    │   Ubuntu machine   │
                    │                    │
                    │    Hermes Agent    │
                    └─────────┬─────────┘
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             ▼                ▼                ▼
        SearXNG         Scrapling MCP    Local Chromium
         Search           Extraction       Interaction
             │
             └───────────────┬───────────────────────┐
                             │                       │
                             ▼                       ▼
                   Focused MCP services       Model inference
                                              │
                              ┌───────────────┼───────────────┐
                              │               │               │
                              ▼               ▼               ▼
                         Windows 11       MacBook Pro       Cloud fallback
                         Ollama            Qwen models      Claude Sonnet
                         Gemma 4 31B       planned
```

The distributed hardware is intentional.

Right now, the Windows 11 workstation is the primary inference node and runs **Gemma 4 31B through Ollama**. Hermes reaches it across the LAN rather than competing with the model for resources on the same machine.

My next step is to make the MacBook Pro another inference node, primarily for **Qwen-family models**. I want to experiment with routing different kinds of work to different models rather than treating one LLM as universally optimal.

That opens the door to a much more interesting design:

```text
Hermes
   │
   ├── Gemma 4 31B
   │      General local reasoning
   │
   ├── Qwen models
   │      Coding / tool-use / specialized workloads
   │
   └── Claude Sonnet
          Fallback
```

Hermes remains the stable control plane while the models underneath it become interchangeable compute resources.

## Local-first does not have to mean local-only

I am also deliberately keeping a cloud fallback.

If the Windows inference machine is offline, Ollama is unavailable, a local model fails badly on a task, or some other part of the local inference path breaks, Hermes can fall back to **Claude Sonnet**.

That may sound contradictory in a project centered around local AI, but I think it is the more practical architecture.

The goal is not ideological purity. The goal is **control over where work goes**.

There is a major difference between intentionally configuring a known fallback and discovering that half of an supposedly local agent's workflow was going through cloud services all along.

My preferred order is:

```text
Local infrastructure first
        ↓
Local alternate model
        ↓
Explicit cloud fallback
```

The important word is **explicit**.

## The hard part is proving the path

The hard part is not wiring these boxes together. It is making sure each request actually takes the route I think it takes.

I want an agent that can search, read, browse, automate, and eventually choose between multiple inference systems without quietly falling back to a gateway I did not intend to use.

So the recurring test is intentionally boring: ask Hermes to search, fetch, browse, and call tools while watching what it actually invokes.

Search should reach SearXNG.

Extraction should reach Scrapling.

Interactive browsing should reach local Chromium.

Specialized workflows should reach their corresponding MCP services.

Inference should reach Ollama on the Windows machine unless another model has been deliberately selected.

And if the local inference path fails, the transition to Claude Sonnet should happen because I configured it that way—not because some hidden dependency made the decision for me.

That observability matters more to me than being able to put a "100% local" badge on the project.

This is still a work in progress, particularly as I add the MacBook as another model host and start experimenting with Qwen alongside Gemma. But the direction is becoming clearer.

The end goal is not one giant AI box.

It is a small, distributed AI system with **Hermes as the control plane, interchangeable models as compute, specialized tools for specific capabilities, and explicit boundaries between local and cloud resources**.

Local means more than where the model runs.

It means knowing where the work goes.
