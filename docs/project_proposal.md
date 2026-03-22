# IoT/Embedded Systems project proposal

<!--toc:start-->

- [Introduction](#introduction)
- [Objectives](#objectives)
- [Materials and Methods](#materials-and-methods)
  - [Devenv](#devenv9)
  - [Opencode](#opencode1)
  - [Openrouter](#openrouter11)
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

In the broader context of Artificial Intelligence (AI), here I refer to an AI
system as an _Agent_ when it is imbued with the capacity to operate autonomously
in a given environment without the need for continuous oversight.

There exist several different pieces of software that enable Agents to operate
in a variety of environments. For programming, Claude Code, released in February
2025, was the first such tool I came to know that allowed AI agents based on a
Large Language Model (LLM) to operate the user's computer autonomously through
its Command Line Interface (CLI). Although it is used, as the name suggests, to
perform coding tasks, this setup allows for LLMs to perform any operation that
can be expressed through CLI commands and outputs. This includes scraping the
web, accessing a local or remote database, reviewing and writing documents in
natural language, etc.

Alternatives to Claude Code soon emerged, including offerings from its main
competitors, such as Gemini CLI and OpenAI Codex, as well as Free and Open
Source (FOSS) alternatives like OpenCode [^opencode] and OpenClaw [^openclaw].

Common to all those tools is the adoption of some open standards for tooling and
configuration that enable more precise control of the agent's capabilities and
behavior, allowing for their optimization to perform specific tasks. For
example, MCP servers allow the user to expose data sources to the agents to be
used as context for their tasks, such as data sources with specialized knowledge
that might not have been part of an agent's pre-training or might conflict with
it, requiring the agent to consider the MCP server as its "source of truth".
Additionally, various forms of prompt injection setups are described in
structured markdown files, informing the agent of specific actions associated
with achieving set goals, usually in a step-by-step manner. Such capabilities
gave rise to various resources and possibilities for the creation of
personalized agents that developers could make use of, which will be discussed
in this document.

## Objectives

This project aims to implement specialized agents that act as natural language
interfaces in an IoT setting. More specifically, it aims to create "research
assistants" that, together, can:

- fetch data from a server, with adequate permissions and correctly constructed
  query statements.

- make use of analytical tools to process fetched data, enabling the extraction
  of insights from it.

- control functionalities programmed into embedded systems, such as activating
  and deactivating sensors or actuators.

- generate reports with supporting graphs and tables, based on their
  interactions with users.

> [!NOTE]
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
  - FOSS LLM models are not considered to be at the _state of the art_.
    According to the _Arena.ai_ leaderboards, which ranks LLMs by having users
    choose the best answer in a blind test between two competing models, Claude
    Opus 4.6 is the current top ranked model for text and code inputs, with a
    calculated Elo rating of 1502 ± 6 [^text-ranking] and 1548 ± 12
    [^text-code], respectively. Claude is followed by several other proprietary
    models. Meanwhile, the top-ranked FOSS model in both these rankings is
    Z.ai's GLM-5 model, with Elo scores of 1455 ± 6 and 1445 ± 10. According to
    the Elo coefficient formula [^elo], a 50-point Elo difference implies the
    higher Elo player should win 57% of matches. However, with a 100-point Elo
    difference (such as seen between Claude and GLM for code inputs) the higher
    Elo player should win 64% of matches. This represents a clear superiority
    often observed between models of different generations (e.g., GPT-3.5
    compared to GPT-4).

- **Pros:**
  - Lower cost: A price comparison per million tokens processed between Claude
    and GLM models on OpenRouter shows Claude costs US\$1.36 for input tokens
    and US\$15.00 for output tokens[^claude-pricing], while GLM costs US\$0.573
    and US\$3.04, respectively[^glm-pricing] (weighted averages across AI
    providers).

  - No risk of vendor lock-in or abusive terms of service: FOSS models such as
    GLM have their source code publicly available [^hf-glm], licensed under the
    standard MIT license, and the rest of the FOSS framework for Agentic AI is
    not bound to any particular LLM model.

  - Stability and reproducibility: Using lock files, such as those generated by
    Nix, we can pin software to specific versions. This ensures consistent
    development environments when building software from source across different
    machines. It also enables rollbacks; even if upgrades introduce breaking
    changes, we can revert to previous stable versions because their source code
    is versioned and available.

  - Extensibility: The project can grow and be extended in various aspects by
    any open-source developer, without being legally bound by any particular
    vendor.

### Devenv[^devenv]

Devenv is a framework for declaratively creating reproducible development
environments using Nix. Nix is a package manager that uses the Nix functional
programming language (of the same name) to install and configure software. It
enables building shell environments, virtual machines, and even entire operating
systems with modularity and compartmentalization. Nixpkgs, Nix's package
repository, is the largest package repository available for Linux systems by
number of packages. With it, we can configure our development environment such
that rebuilding from scratch is as simple as running `devenv shell` in the
project repository.

> [!NOTE]
>
> Nix is how I manage my own computer's configuration [^nix-config]

### Opencode[^opencode]

Client and framework for accessing, configuring, and interacting with AI Agents
through terminal and web-based interfaces. This is the glue that will bind the
components of our project.

> [!NOTE]
>
> Openclaw[^openclaw] is an alternative framework, very popular in China, that
> allows interaction with agents over a variety of messaging apps, such as
> WhatsApp, Signal, and others.

### Openrouter[^openrouter]

OpenRouter provides a unified interface to connect to various AI providers,
allowing us to fetch LLM models.

#### Resources

- Promptfoo [^promptfoo]: a tool to programmatically test and compare how
  different models perform when responding to a given set of prompts.

### Model Context Protocol (MCP)

Open standard for providing context to LLM conversations, allowing better access
to correct and up‑to‑date information about external tools, systems, and data
sources without relying exclusively on its pre‑training [^mcpwen]. This resource
can enable even relatively small models to perform on par with larger, more
general‑purpose models. Many pre‑configured local or remote MCP servers are
available, such as:

#### Resources

- GitHub's Official Registry [^mcp-github].

- Smithery [^mcp-smithery].

- Awesome MCP Servers [^mcp-servers].

- OpenCode MCP Documentation [^mcp-opencode].

### Language Server Protocol (LSP) and Formatters

These are tools to verify that code is syntactically correct (along with other
checks like missing libraries or function definitions) and well‑formatted
according to given standards. They are essential for enabling agents to write
code effectively. OpenCode already includes various pre‑installed and
pre‑configured LSP servers, but we may want to add others or adjust default
settings.

#### Resources

- LSP Specification [^lsp-org].

- OpenCode Documentation [^lsp-opencode].

### Skills (Prompt Injection)

These are Markdown‑formatted documents containing instructions for AI models to
perform specific tasks effectively.

#### Resources

- anthropics/skills [^skills-antropic].

- openai/skills [^skills-openai].

- Tech Leads Club's Agent Skills [^skills-tlc].

- hoodini/ai-agents-skills [^skills-houdini].

- skills.sh [^skills-sh].

- OpenViking: A tool for hierarchical skill management that reduces token
  consumption. [^skills-viking]

### Tools

These encompass external resources that agents interact with (e.g., SQL servers,
Python environments with scientific computing libraries). These should be
exposed and configured with appropriate permissions.

#### Resources

- OpenCode Documentation on Tooling [^tools-opencode].

### Docusaurus

Docusaurus is a tool for generating wikis from markdown‑formatted files. It will
be used to document this project, the created agents, and their configurations.

#### Resources

- Docusaurus Documentation [^docs-docusaurus]

- LazyVim Documentation: A well‑formatted reference on constructing LazyVim
  configurations for NeoVim, which could serve as a model for our documentation.
  [^docs-lazyvim]

- ICMC Processor: An example of documentation I created. [^docs-icmc]

### Preexisting Agent configurations

These are agents created and shared by other authors.

#### Resources

- The Agency [^agents-agency].

- oh-my-opencode [^agents-omo].

- opencode-agents [^agents-opencode].

- opencode-workspace [^agents-workspace].

- Opencode documentation on configuring agents [^agents-doc].

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
