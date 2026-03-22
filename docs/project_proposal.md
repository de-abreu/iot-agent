# IoT/Embedded Systems project proposal

<!--toc:start-->

- [Introduction](#introduction)
- [Objectives](#objectives)
- [Materials and Methods](#materials-and-methods)
  - [Devenv](#devenvdevenv)
  - [Opencode](#opencodeopencode)
  - [Openrouter](#openrouter-openrouter)
    - [Resources](#resources)
  - [Model Context Protocol (MCP)](#model-context-protocol-mcp)
    - [Resources](#resources-1)
  - [Language Server Protocol (LSP) and Formatters](#language-server-protocol-lsp-and-formatters)
    - [Resources](#resources-2)
  - [Skills (Prompt Injection)](#skills-prompt-injection)
    - [Resources](#resources-3)
  - [Tools](#tools)
    - [Resources](#resources-4)
  - [Docusaurus](#docusaurus)
    - [Resources](#resources-5)
  - [Preexisting Agent configurations](#preexisting-agent-configurations)
    - [Resources](#resources-6)
- [Cronogram](#cronogram)
- [References](#references)

<!--toc:end-->

## Introduction

In the broader context of Artificial Inteligence (AI), here I refer to an AI
system as an _Agent_ when it is imbued with the capacity to operate autonomously
in a given environment without the need for continuous oversight.

There exits several different pieces of software that enable Agents to operate
in a variety of environments. For programming, Claude Code, released in February
2025, was the first such tool I came to know that allowed AI agents based on a
Large Language Model (LLM) to operate the user's computer autonomously though
its Command Line Interface (CLI). Though it is used to, as the name suggests,
perform coding tasks, this setup allows for LLMs to perform any operation that
can be expressed through CLI commands and outputs. This includes scraping the
web, accessing a local or remote database, reviewing and writing documents in
natural language, etc.

It did not take long for alternatives to Claude Code to emerge. Be it from its
main competitors, such as Gemini CLI and OpenAI Codex, be it Free and Open
Source (FOSS) alternatives, such as OpenCode [^opencode] and OpenClaw
[^openclaw].

Common to all those tools is the adoption of some open standards for tooling and
configuration that enable more precise control of the agent's capabilities and
behavior, allowing for their optimization to perform specific tasks. For
example, MCP servers allow the user to expose data sources to the agents to be
used as context for their tasks, such data sources with specialized knowledge
that might not have been part of the agent's pre-training, or might be conflict
with it, and requires the agent to assume the MCP server as its "source of
truth"; Also, various forms of prompt injection setups described in structured
markdown files to inform the agent of specific actions associated with achieving
goals set, usually in a step-by-step manner. Such capability gave rise to
various resources and possibilities for the creation of personalized agents that
developers could make use of, which will be discussed here.

## Objectives

The implementation of specialized agents to act as a natural language interface
in the context of an IoT setting. More specifically, it aims to create "research
assistants" that, together, are able to:

- fetch data from a server, with adequate permissions and with a correct
  construction of query statements.

- make use of analytical tools to process fetched data, enabling the extraction
  of insights from them.

- operate functionalities programmed into the embedded systems, such as the
  activation and deactivation of sensors or actuators.

- generate reports with supporting graphs and tables, based on its interaction
  with the user.

> [!note]
>
> This is a description of the project still in general terms, so as to give an
> idea of what it intends to achieve. In a conversation with Sebastian he has
> told me of his ongoing project on the topic of air pollution monitoring, which
> could benefit from the use of AI agents. If we were to implement this solution
> upon its existing infrastructure, we could narrow down the objectives further
> to meet actual requirements, such as: communication with a specific server
> architecture, use of a given set of analytical tools, existing functionality
> in embedded systems, and so on.

## Materials and Methods

For this project, I intend to make use solely of FOSS tools. There are pros and
cons associated with that choice:

- **Cons:**
  - There are no FOSS LLM models are not considered to be at the _state of the
    art_. According to the _Arena.ai_ leaderbords, that ranks LLMs by having
    users choose the best answer in a blind test between two competing models,
    Claude Opus 4.6 is the current top ranked model for text and code inputs,
    with an calculated Elo of 1502 ± 6 [^text-ranking] and 1548 ± 12
    [^text-code], respectively. Claude is followed by several other models that
    are also proprietary. Then the top ranking FOSS model in both these rankings
    is Z.ai's GLM-5, with Elo scores of 1455 ± 6 and 1445 ± 10. According to
    Elo's coefficient formula [^elo], a 50 point Elo difference implies the
    higher Elo player should win 57% of the matches played. But when you get to
    100 point Elo difference (such as seen between Claude and GLM for code
    inputs) the higher Elo player should win 64% of the matches played. That is
    a clear superiority often seen between models of different generations (say,
    GPT-3.5 when compared to the newer GPT-4).

- **Pros:**
  - Lower cost: In a price comparison per million tokens processed between the
    previously mentioned Claude and GLM models in OpenRouter, Claude costs US\$
    1.36 on input tokens and US\$ 15.00 on output tokens[^claude-pricing]. While
    GLM only costs US\$ 0.573 and US\$ 3.04, respectively[^glm-pricing] (these
    are weighted averages of the various AI providers that offer these models).

  - No risk of incurring in vendor lock-in or abusive Terms of Service: FOSS
    models such as GLM have their source code publicly available [^hf-glm],
    licensed using the standard MIT license, and the rest of the FOSS framework
    for Agentic AI is not bound to use any particular LLM model.

  - Stability and reproducibility: With lock files, such as those generated
    using Nix, we can bind our project to build software at specific versions.
    This ensures we get the same development environment when building software
    from source at different machines. It also allows rollbacks, so that even
    though there might be upgrades to those that introduce breaking changes, we
    can always build from previous, stable version since its versioned source
    code is available.

  - Extensibility: The project can grow and be extended in its various aspects
    by any open-source developer, without being legally bound by any particular
    vendor.

### Devenv[^devenv]

Framework for declaratively creating reproducible development environments using
Nix. Nix is a package manager that allows for the installation and configuration
of software using the (also named) Nix functional programming language. It
allows one to build shell environments, virtual machines and even an OS with
modularity and compartmentalization. Nixpkgs, the package repository used by
Nix, is the largest package repository by number of packages available to all
Linux systems. With it we can configure our developer environment in such a way
that rebuilding it from scratch should be as simple as running the command
`devenv shell` on the project's repository.

> [!NOTE]
>
> Nix is how manage my own computer's configuration [^nix-config]

### Opencode[^opencode]

Client and framework for accessing, configuring, and interacting with AI Agents
though terminal and web-based interfaces. This is the glue that will bind the
components of our project.

> [!NOTE]
>
> Openclaw[^openclaw] is an alternative framework, very popular in China, that
> allows the interaction with the Agent to happen over a variety of messaging
> apps, such as Whatsapp, Signal, among others.

### Openrouter [^openrouter]

Unified interface to connect to various AI providers, this is where we can fetch
our LLM models.

#### Resources

- Promptfoo [^promptfoo]: a tool to programatically and comparatively test how
  different models perform when responding to a given set of prompts.

### Model Context Protocol (MCP)

Open standard for provisioning context to the conversation with the LLM model,
allowing it to have better access correct and up to date information on external
tools, systems, and data sources without relying exclusively in it pre-training
[^mcpwen]. This resource can be used to allow even relatively small models to
have a performance on par with bigger, more general use models. There are many
MCP resources for pre-configured local or remote MCP servers, such as:

#### Resources

- Github's Official Registry [^mcp-github];

- Smithery [^mcp-smithery];

- Awesome MCP Servers [^mcp-servers];

- OpenCode MCP Documentation [^mcp-opencode]

### Language Server Protocol (LSP) and Formatters

Respectively, software to verify that a given code is syntactically correct
(plus other checks such as missing libraries or function definitions) and well
formatted by given standards. An absolute necessity for enabling Agents to
better write code. OpenCode already ships with a variety of LSP srevers
pre-installed and pre-configured but we might want to add others or change some
of the defaults settings.

#### Resources

- LSP Specification [^lsp-org]

- OpenCode Documentation [^lsp-opencode]

### Skills (Prompt Injection)

Markdown formatted documents with instructions for the AI model to perform given
specific tasks effectively.

#### Resources

- anthropics/skills [^skills-antropic]

- openai/skills [^skills-openai]

- Tech Leads Club's Agent Skills [^skills-tlc]

- hoodini/ai-agents-skills [^skills-houdini]

- skills.sh [^skills-sh]

- Openviking: A tool to manage skills hierarchically, lowering token
  consumption. [^skills-viking]

### Tools

The many external resources that our agent should act upon (say, sql servers,
python environment with scientific computing libraries). Preferably exposed and
configured with proper permissions settings.

#### Resources

- OpenCode Documentation on Tooling [^tools-opencode]

### Docusaurus

A tool to generate wikis from a repository of markdown formatted files. Should
be used to document this project, the agents created and their specific
configurations.

#### Resources

- Docusaurus Documentation [^docs-docusaurus]

- LazyVim Documentation: A very well formatted documentation on how to the
  LazyVim configuration for NeoVim is constructed, could be used as a reference
  for our own documentation. [^docs-lazyvim]

- ICMC Processor: An exemplo of documentation of my own creation. [^docs-icmc]

### Preexisting Agent configurations

Agents created and shared by other authors.

#### Resources

- The Agency [^agents-agency]

- oh-my-opencode [^agents-omo]

- opencode-agents [^agents-opencode]

- opencode-workspace [^agents-workspace]

- Opencode documentation on configuring agents [^agents-doc]

## Cronogram

> [!WARNING]
>
> TBD

## References

[^opencode]: https://opencode.ai/

[^openclaw]: https://openclaw.ai/

[^text-ranking]: https://web.archive.org/web/20260322130151/https://arena.ai/leaderboard/text

[^text-code]: https://web.archive.org/web/20260314042745/https://arena.ai/leaderboard/code

[^elo]: https://en.wikipedia.org/wiki/Elo_rating_system

[^claude-pricing]: https://web.archive.org/web/20260322095421/https://openrouter.ai/anthropic/claude-sonnet-4.6/pricing

[^glm-pricing]: https://web.archive.org/web/20260322095401/https://openrouter.ai/z-ai/glm-5/pricing

[^devenv]: https://devenv.sh/

[^nix-config]: https://github.com/de-abreu/nix-config

[^openrouter]: https://openrouter.ai/

[^promptfoo]: https://promptfoo.dev/

[^mcpwen]: https://modelcontextprotocol.io/

[^mcp-github]: https://github.com/modelcontextprotocol/servers

[^mcp-smithery]: https://smithery.ai/

[^mcp-servers]: https://github.com/wong2/awesome-mcp-servers

[^mcp-opencode]: https://opencode.ai/docs/

[^lsp-org]: https://microsoft.github.io/language-server-protocol/

[^lsp-opencode]: https://opencode.ai/docs/

[^skills-antropic]: https://github.com/anthropics/skills

[^skills-openai]: https://github.com/openai/skills

[^skills-tlc]: https://github.com/tech-leads-club/agent-skills

[^skills-houdini]: https://github.com/hoodini/ai-agents-skills

[^skills-sh]: https://skills.sh/

[^skills-viking]: https://github.com/volcengine/OpenViking

[^tools-opencode]: https://opencode.ai/docs/tools/

[^docs-docusaurus]: https://docusaurus.io/

[^docs-lazyvim]: https://lazyvim.github.io/

[^docs-icmc]: https://de-abreu.github.io/Processador-ICMC/en/

[^agents-agency]: https://github.com/msitarzewski/agency-agents

[^agents-omo]: https://github.com/code-yeongyu/oh-my-opencode

[^agents-opencode]: https://github.com/Shakudo-io/opencode-agents

[^agents-workspace]: https://github.com/kdcokenny/opencode-workspace

[^agents-doc]: https://opencode.ai/docs/agents/

[^hf-glm]: https://huggingface.co/zai-org/GLM-5
