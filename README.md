# DeepSeek Harness In-Depth Analysis

**Everything is a plugin.**

A sixteen page architecture analysis of **DeepSeek Harness 0.1.7-rc.2** (`dsh`), drawn from the source code as it is
built. Nothing on the pages is aspirational. Every block cites the repository file, line and exact quote it comes
from, and all 444 of those citations are in [`citation_register.csv`](citation_register.csv).

Source snapshot: 25 September 2026, 13850 repository files.

![The overview page, every connector flowing from its source to its target](DeepSeek%20Harness%20Overview.gif)

*The overview page (E0). Every connector flows from its source to its target: a caller starts a turn, the turn
moves through six numbered stages, and the result is logged and fed back to the model.*


## What is in this repository

* **[`DeepSeek Harness Overview.gif`](DeepSeek%20Harness%20Overview.gif)**: the overview page (E0), animated.
* **[`DeepSeek Harness Architecture_As Built.drawio`](DeepSeek%20Harness%20Architecture_As%20Built.drawio)**: all
  sixteen pages, editable. Zone titles and page chips are clickable links between pages.
* **[`DeepSeek Harness Architecture_As Built.pdf`](DeepSeek%20Harness%20Architecture_As%20Built.pdf)**: the print
  version, one page per diagram.
* **[`citation_register.csv`](citation_register.csv)**: every block on every page, with its citation id, label, the
  pages it appears on, npm package, repository path, context key, document, maturity state and evidence (file and
  line).
* **[`png/`](png)**: one image per page, named in page order.
* **[`LICENSE`](LICENSE)**: Creative Commons Attribution 4.0 International.

**Opening the drawio file:** use the draw.io desktop app or [app.diagrams.net](https://app.diagrams.net) (File, Open
from, Device). The page tabs run along the bottom. The overview page is drawn on a 2560 by 1440 canvas, every other
page on 1920 by 1080, all in the same 16:9 shape.


## The story in one paragraph

DeepSeek Harness is an agent harness that runs as **one Node host process on your own machine**, and it is built so
that **every part is a plugin**, including the agent loop, the model adapter and the tool registry. A caller (the web
app, the desktop app, a one shot command, an SDK program or an ACP editor) launches a named profile. The profile
composes a plugin tree from bundle patches. A turn then enters an inbox, the harness assembles what the model sees,
and the agent loop runs one model call plus its tools per step. Every tool call passes a guarded pipeline with hooks,
approval and guards, and runs in an execution world confined by an operating system sandbox. **Every step is written
to an append only session log**, and the history the model sees is derived from that log, never kept on the side.
DeepSeek is the default model provider, reached over HTTPS. OpenAI and Anthropic compatible endpoints are reachable
through a second adapter.

The project states five design rules, and the pages show where each one lives in the code:

* Everything is a plugin
* Model visible means logged
* Plugins, not loop changes
* Misconfiguration fails loud
* Explicit over implicit


## How to read the diagrams

**Layer colours.** Each colour means the same thing on every page.

<table>
<tr><td><b>A</b> Execution world</td><td><b>B</b> External services</td><td><b>C</b> Protocols and API</td><td><b>D</b> Session data</td></tr>
<tr><td><b>E</b> Model layer</td><td><b>F</b> Harness core</td><td><b>G</b> Security and observability</td><td><b>H</b> People and surfaces</td></tr>
</table>

**Maturity markers.**

* A solid block is shipped and on by default.
* A solid block with a grey **Off** chip is shipped, but off by default or opt in.
* A slate dashed block is an experimental package.

**Chips and marks.**

* **Citation chip** (for example `C121`, top right of a block): look the id up in the citation register. Citation
  chips are hidden on the overview page, where every block is still a cited block.
* **Tag chips** inside the bottom of a block name the exact system or standard used, for example bubblewrap, SQLite
  FTS5 or MCP SDK 2.0.0. A version appears only where the repository pins it.
* **Numbered circles** mark a real ordered sequence in the code: a turn, the tool pipeline, the model request path,
  approval, composition order, the plugin lifecycle.
* **Diamonds** are real decision logic, such as which launch profile or which sandbox backend.
* **Protocol pills and link colours**: MCP (teal), ACP (magenta), JSON RPC (gold), Remote (slate), Trust (green),
  OTLP (purple).
* **Right column notes** are verified statements, each with its own evidence.
* **Page chips** (W1, W2 and so on) name the page that tells that part in full, and open it in the drawio file.


## Page by page

### E0 Overview: DeepSeek Harness, everything is a plugin

![E0 Overview](png/01%20E0%20Overview.png)

**The question it answers:** how does one turn move through the whole harness, and which page tells each part in full?

**What it shows.** The page follows one turn from left to right.

* **Callers**, on the left, each paired with the named profile it launches. Below them is the path from the browser
  page to the host: remote calls, the stream socket, the web server on port 3080, the client connection and the
  desktop host on port 19387. Three coloured links carry the wire protocols into the harness: Remote, JSON RPC and
  ACP.
* **The harness**, inside a dashed red boundary, runs six numbered stages:
  1. **Inbox**: follow up messages, steering input, injected context and webhook deliveries (off by default).
  2. **Model context**: the system prompt, runtime context, skills and instructions, file and session references,
     and the model history derived from the log.
  3. **Agent loop**: the model service (dark) and the tools (red) trading tool calls and results. One model call and
     its tools make a step, with up to 10 tool calls at once.
  4. **Tool gate**: the "Allowed?" diamond, with pre execute hooks, approval and guards beneath it.
  5. **Execution world**: shell, files, search, code, jobs, web, MCP, browser use and remote SSH, all confined by
     the sandbox.
  6. **Result logged, back to the model**: the loop that closes every step.
* **Above the stages** sits what shapes a turn: the agent presets, compaction (with its "should compact?" link to
  the context), the model request path, and delegation, drawn as a stack because up to 8 children can run at once.
* **Below the stages** runs the session log. Dashed orange links show what each stage writes. The one link going up
  shows that model history is derived from the log.
* **Beyond the machine boundary**, on the right: model providers (DeepSeek API and account, compatible endpoints, your
  own endpoint) and other external services (web search providers, MCP servers, GitHub, npm, Codex and Claude Code).
* **The top band** shows how the plugin tree is composed at launch, in the order bundle patches, profile patch, home
  patch, command line overlays, telemetry switch. It also shows how a plugin is installed, peer checked, loaded, hot
  reloaded and unloaded.
* **The bottom band** holds evaluation and releases, security and trust, and the capability seams. Two blue dashed
  links run from security into the flow: **Enforce** into the tool gate, **Limit** into the execution world.
* **The page guide** at the foot says in one line why each of the other pages exists.

**Key takeaways.**

* The turn is a loop, not a pipeline. A step ends by logging its result and handing it back to the model.
* The session log is the spine. What the model sees can always be rebuilt from it.
* Security reaches into the turn at two points on this page: approval enforced at the tool gate, and the sandbox
  limiting the execution world.
* The project describes itself as a developer preview that has not had a security audit.


### W0 Master: DeepSeek Harness reference architecture

![W0 Master reference architecture](png/02%20W0%20Master%20reference%20architecture.png)

**The question it answers:** what is every part of the web profile, and how does each part connect?

**What it shows.**

* **Callers**: web app (browser UI on port 3080), desktop app (Electron 44), one shot command line, SDK programs in
  TypeScript and Python over stdio, and ACP clients.
* **The DeepSeek Harness host**, one Node process built from plugins on Cordis 4.0.4:
  * the plugin tree and the web server and API gateway;
  * the agent harness: agent loop, agent registry, model selection, system prompt, context compaction, tool registry,
    subagents, workflow engine, and plan, goals and todos;
  * session and context: session log, session search (off), skills and instructions;
  * the execution world: shell, subprocesses, file system, code runtime, background jobs, web search and fetch,
    terminal tabs, language servers (off), remote over SSH (off), and browser and computer use (experimental);
  * the agent presets: Standard, PTC, Minimal, Creator, and Agent Teams (experimental).
* **Protocols and storage**: caller protocols (Remote API, SDK server, ACP server), model adapters (DeepSeek, pi ai,
  retry policy), tool protocols (MCP client, hook bridges, GitHub webhook) and local storage (session files, key value
  storage, attachments).
* **The machine boundary**, and beyond it the model providers and other external services.
* **The security, observability and governance strip**: sandbox, write fence, approval, permission presets, loop
  guards, credential store, launch token and cookie, session telemetry, runtime invariants.

**Key takeaways.**

* One host process on your machine holds everything. W4 lists exactly what leaves it: model requests, the session
  log added to DeepSeek requests, and telemetry after feedback.
* The host is not a monolith. The gateway, the agent services and the execution world are each a group of plugins.
* The SDK and ACP servers, the MCP client and the hook bridges ship, but are off in the web profile.


### W1 Agent harness internals

![W1 Agent harness internals](png/03%20W1%20Agent%20harness%20internals.png)

**The question it answers:** what exactly happens inside one turn, and which services does every agent reuse?

**What it shows.**

* **Where a turn starts**: one inbox fed by follow up messages, steering input, injected context and webhook
  deliveries.
* **One turn, step by step**, in ten numbered steps: turn start, claim input, assemble prompt, pre step decision,
  prepare the call, freeze history, stream the reply, run tool calls, step end, turn stopping.
* **The tool pipeline** every call follows, in seven steps: call logged, pre execute, approval, guards, execute
  wrappers, post execute, result logged.
* **The model call**: request series, retry policy, cancellation, token meter.
* **Core services** built once and reused by every agent, the **extension points** where new behavior attaches
  (session, agent and capability events, and waterfalls where returning early stops the chain), and the **limits and
  guards** that keep a runaway loop in check.

**Key takeaways.**

* A step is one model request plus the tools it calls. A turn is zero or more steps.
* Anything that reaches a model request must be rebuildable from the session log.
* New behavior goes on documented extension points. Changing the loop means updating the architecture document.
* A failed model call is retried inside the same open turn, over the same logged history.
* Repeat call reminders come after 3, 5 and 8 identical calls, and up to 10 tool calls run in parallel in a step.


### W2 Session log, context and memory

![W2 Session log, context and memory](png/04%20W2%20Session%20log,%20context%20and%20memory.png)

**The question it answers:** what does the log record, how is it stored and read, and what reaches the model?

**What it shows.**

* **What the log records**, one row per event: turns and steps, messages, tool calls and results, request headers
  and context, failed attempts and plugin events.
* **On disk**: a session folder under DSH_HOME, a versioned JSONL file (format version 4), Zstandard compressed
  frames, a write lease and atomic publish.
* **Format versions**: four adjacent migrations, one package each.
* **Reading the log**: model history, session projections, a projection cache, session stats and a turn outline, all
  derived views and never a second store.
* **Context the model sees**, **compaction** (a pressure trigger at 80 percent of the window, the last 16 percent kept
  verbatim, a logged summary), **large tool output** (inline up to 12500 tokens, then a private spill store), **other
  stores**, and **memory through MCP** (reference configurations, off by default).

**Key takeaways.**

* The session log is the source of the context the model sees. Model history is derived from it.
* Committed files are never renamed, replaced or deleted. A write publishes the next version beside them.
* Compaction is logged like everything else, as a summary that replaces the compacted range.
* Session search indexes past sessions with SQLite FTS5, and is off by default.


### W3 Tools and integrations

![W3 Tools and integrations](png/05%20W3%20Tools%20and%20integrations.png)

**The question it answers:** what can the model call, and how does the harness reach other systems?

**What it shows.**

* **Work in the workspace**: shell command (bash or PowerShell), read, write and edit, glob and grep (packaged
  ripgrep), run code, persistent shell, terminal tools, background jobs, office documents.
* **Plan and coordinate**: exit plan mode, goals, todo list, ask the user, present files, load a skill, schedule a
  message, inspect the plugin tree.
* **Delegate**: subagent, fork, message and interrupt, workflow, Ralph loop, and Codex and Claude Code as subagents.
* **Look things up**: web search (DeepSeek by default, with Exa and Perplexity as opt in providers), web fetch, MCP
  resources, session history, code navigation.
* **The MCP client**, **other protocols** (ACP server, ACP subagent, SDK server, hook bridges, GitHub webhook),
  **when a tool asks first**, **browser and computer use** (experimental providers) and **remote over SSH**.

**Key takeaways.**

* Seventy tool entries ship in the repository, 65 distinct tool names.
* In the web profile the tools come from the chosen agent preset.
* MCP tools are named after their server, so session history and permission rules survive restarts.
* Over SSH only files, processes and the sandbox move. The harness, model calls and sessions stay local.


### W4 Security, trust and governance

![W4 Security, trust and governance](png/06%20W4%20Security,%20trust%20and%20governance.png)

**The question it answers:** what confines the agent, who approves wider access, where do secrets live, who may call
the host, and what leaves the machine?

**What it shows.**

* **Sandbox modes**: read only, workspace write (the default), full access, and a fail closed state when no backend
  is usable.
* **Sandbox backends**, split by platform: bubblewrap and Landlock on Linux, Seatbelt on macOS, a restricted token on
  Windows, and a remote sandbox over SSH.
* **Approval** in five ordered steps, the **approval channels** (web approval card, ACP client answers) and the
  **permission presets** (read only, workspace, full access, and an experimental auto review).
* **Credentials and identity**, **web access control** (loopback bind, launch token, signed cookie, host and origin
  checks, a 300 MiB body cap) and **process and file hygiene**.
* **What leaves the machine**, and the **stated safety posture** in the project's own words.

**Key takeaways.**

* The sandbox confines what commands may write, and each call reports whether enforcement was full or partial.
* Full access skips the sandbox, and the approval policy then becomes never.
* An unanswered approval defaults to unavailable, which denies the call.
* Every approval is written to the session log as an asked and decided pair.
* Official DeepSeek requests carry a session log suffix by default. Telemetry is sent only after feedback.


### W5 Observability and evaluation

![W5 Observability and evaluation](png/07%20W5%20Observability%20and%20evaluation.png)

**The question it answers:** what does the harness report while it runs, and how is quality proved before release?

**What it shows.**

* **Session telemetry** (OpenTelemetry, OTLP log export, a redaction step), **feedback and export** (feedback
  command, message ratings, session export) and **runtime checks** (runtime invariants, token meter, console logger,
  inspector).
* **Test tiers**: unit tests on Vitest, a coverage gate, real API tests, snapshot replay, and lint, type and
  duplication checks.
* **Six benchmark gates**, **CI on GitHub**, **repository gates** and four written **postmortems**.

**Key takeaways.**

* Telemetry releases the session log only after explicit user feedback, and an environment variable turns it off.
* Coverage is gated at 100 percent per file for package sources.
* Every non trivial user visible change updates a recorded session snapshot.
* Benchmark timing runs in workers under plain Node, against built packages.


### W6 Runtime topology and decisions

![W6 Runtime topology and decisions](png/08%20W6%20Runtime%20topology%20and%20decisions.png)

**The question it answers:** which profile launches, what runs in which process, which sandbox backend is chosen, and
where does data live?

**What it shows.**

* **The launch decision**, as a row of diamonds: a person in a browser gets the web profile, the desktop app the
  desktop profile, one task with no interface the headless profile, an ACP client the ACP profile, and an SDK program
  the SDK profile, or SDK minimal without the base bundle.
* **The sandbox decision**: danger full access means no confinement, Windows uses its ACL runner, macOS its Seatbelt
  profile, and Linux tries bubblewrap before the Landlock launcher.
* **Processes**, the **data home** (DSH_HOME, profiles, credentials, session files), **one execution world** and
  **platforms and packaging**.

**Key takeaways.**

* Every supported Node application launches through a named dsh profile.
* Desktop runs the same web stack in its own host process, with its own reserved profile.
* File and subprocess providers share one execution world, so a remote provider moves shells, terminals and language
  servers together.
* When no backend on the host is usable, a confined mode fails closed instead of running the command unconfined.


### W7 Agent catalog and worked flow

![W7 Agent catalog and worked flow](png/09%20W7%20Agent%20catalog%20and%20worked%20flow.png)

**The question it answers:** what agents can you pick, how does work get delegated, and what really happens when a
command needs approval?

**What it shows.**

* **Four agent presets**: Standard (the default, full tool set), PTC (tools called in batches from code), Minimal
  (only a terminal tool) and Creator (the agent writes plugins for DSH).
* **Subagent providers** (spawn, fork, ACP, Codex, Claude Code, a DSH SDK child), **continuation and limits** (up to 8
  active children, one level deep by default) and **Agent Teams** (experimental).
* **A worked flow** traced through the code in nine steps: the model calls bash, pre execute passes, the command runs
  confined, the write is denied, the model asks for more, approval is asked, it is allowed once, the command runs with
  wider access, and the result is logged.
* **What the flow writes** to the session log, and the **workflow engine**.

**Key takeaways.**

* The retry must be the exact same command, once, in the same turn, with a one sentence justification.
* A child joins the parent's exact preset revision, then gets a delegation context and a tool filter.
* Only the Lead creates or interrupts teammates.
* A preset can be chosen only before the first turn of a session.


### W8 Lifecycle and operating model

![W8 Lifecycle and operating model](png/10%20W8%20Lifecycle%20and%20operating%20model.png)

**The question it answers:** how is a running tree composed and changed, and how is the project built and released?

**What it shows.**

* **Composition order**, where later layers win: bundle patches, profile patch, home patch, command line overlays,
  telemetry switch. Then the **patch rules**.
* **The plugin lifecycle**: install a bundle (pnpm 11.7.0), peer check, load into the tree, hot reload, unload, and
  the **install safeguards** around it.
* **Before a change merges**, the **release families** (npm packages, vendored Cordis, native addon, Python wheels,
  desktop installers), **desktop updates** and the **session format lifecycle**.

**Key takeaways.**

* A patch replaces a row's whole config. It never merges into it.
* Enabling a bundle appends it at the end of the list, which can change which layer wins.
* Migrations never move, overwrite or delete a released session file.
* Desktop clients only accept a greater version, so a feed cannot move an install backwards.


### W9 Profile and bundle composition

![W9 Profile and bundle composition](png/11%20W9%20Profile%20and%20bundle%20composition.png)

**The question it answers:** what does each shipped profile mount, and what does each agent preset add?

**What it shows.**

* **An upper matrix**: fifteen capabilities (agent loop, adapters, session files, sandbox mode, approval, tools,
  compaction, telemetry, runtime invariants, hot reload, surface) across the six profiles: Web, Desktop, Headless,
  SDK, ACP and SDK minimal.
* **A lower matrix**: the four web agent presets against the shell, file, web, subagent, workflow, code mode and
  plugin manager tools.

**Key takeaways.**

* Every cell is read from the bundle patch files, composed in profile order.
* In web and desktop the tools live in the agent presets, so those host rows read "Per preset".
* SDK minimal is one standalone bundle, and it pins full access.
* Desktop reuses the web bundles under its own profile name.
* "Platform gated" means bash or PowerShell, depending on the operating system.


### W10 Client and host topology

![W10 Client and host topology](png/12%20W10%20Client%20and%20host%20topology.png)

**The question it answers:** how do the browser page, the host process, the desktop shell and outside programs talk
to each other?

**What it shows.**

* **The browser page**, a React app on the loopback host: client kernel, client store, conversation, sidebars and
  settings.
* **The wire** between page and host: remote calls, the stream socket (ws 8.21), client plugin bundles, dev reload
  events.
* **The host process on port 3080**, loopback only, one operator.
* **The desktop app**: the Electron main process, the app scheme, cookies, browser tabs, auto updates, and the host
  child on port 19387.
* **Stdio clients** that spawn separate SDK and ACP profiles.

**Key takeaways.**

* Remote calls are HTTP POST requests, and streams share one WebSocket.
* Desktop serves the page from its own scheme and forwards API calls to its host on port 19387.
* The page authenticates with a signed HttpOnly cookie, issued against the launch token.
* SDK and ACP clients spawn dsh and speak JSON RPC over its standard input and output.


### W11 Model layer

![W11 Model layer](png/13%20W11%20Model%20layer.png)

**The question it answers:** how does a request reach DeepSeek and come back, and where is the model chosen?

**What it shows.**

* **The model seam**, where every provider connects, and the credentials lookup.
* **The model request pipeline** in seven steps: build request, resolve route, send the request, stream the reply,
  retry on failure, meter tokens, append to the session log.
* **The DeepSeek endpoint and adapter** (official and account routes, the Anthropic Messages wire format, the default
  model catalog with DeepSeek flash and V4 Pro) and **the pi ai adapter** (OpenAI completions and responses,
  Anthropic messages, self hosted endpoints).
* **Where a model is chosen**: the session default, the subagent route, the compaction policy and the session title
  model, four independent picks.

**Key takeaways.**

* Every request carries the session id in a header, and the turn history becomes its Messages fields.
* Extension fields that plugins add are committed only after the response succeeds.
* Retries cover rate limit, server, timeout and transport errors, not every failure.
* The pi ai adapter is kept beside the DeepSeek adapter as a design verification twin.


### S1 Capability seam map

![S1 Capability seam map](png/14%20S1%20Capability%20seam%20map.png)

**The question it answers:** which capabilities are swappable, and who defines, provides and consumes each one?

**What it shows.** The first 17 of the 33 capability seams, each as a row with four columns: the service definition,
the provider mounted by a shipped profile, other providers in the repository, and the direct consumers. Rows include
MCP resources, browser use, computer use, attachments, the model service, session persistence, credentials,
authorization, session telemetry, storage, session query and skills.

**Key takeaways.**

* A seam is a swappable capability with three roles: a service definition, a provider and a consumer.
* One provider swap changes the whole product, which is why these seams exist.
* A logo on a row names its shipped default, for example the OpenTelemetry SDK or SQLite FTS5.


### S1B Capability seam map, continued

![S1B Capability seam map, continued](png/15%20S1B%20Capability%20seam%20map,%20continued.png)

**The question it answers:** the same question, for the remaining 16 seams.

**What it shows.** Subprocess, shell, terminals, sandbox, approval, code runtime, file system, compaction, subagents,
speech to text (experimental), background jobs, web search and fetch, spill store, directory picker, workflow engine
and language servers.

**Key takeaways.**

* A capability seam is complete only with all three roles.
* On Linux the sandbox tries bubblewrap first, then Landlock through a bundled native launcher.
* The PowerShell sandbox also registers the shell service on Windows, a provider the repository's own docs table
  leaves out.


### S2 Legend and citation register

![S2 Legend and citation register](png/16%20S2%20Legend%20and%20citation%20register.png)

**The question it answers:** how do I read every mark on these pages, and where does each block come from?

**What it shows.** The maturity markers, layer colours, links and protocols, line styles, and chips and badges used
across the set, next to the first rows of the citation register. The full register, 444 citations, is
[`citation_register.csv`](citation_register.csv).


## Counted facts

These numbers are derived again from the repository on every build of the pages.

* 70 tool entries, 65 distinct tool names
* 33 capability seams
* 14 core session event types, session format version 4
* 6 benchmark gates
* Web server on port 3080, desktop host on port 19387
* Up to 8 active subagents, one level of delegation by default
* Up to 10 tool calls in parallel within a step


## How the analysis was sourced

* Only what is built in the repository is drawn. Nothing was added to make a page look complete, and absent
  features are never mentioned.
* Where the docs and the code disagree, the pages follow the code.
* Every block carries evidence: a file, a line and an exact quote. A citation gate reads the repository again and fails
  the build when a quote is not within two lines of its cited line, when a block has only documentation evidence
  without a reason, when a version's digits are not in the evidence, or when a maturity state disagrees with the
  composed profile state.
* Profile states come from composing the bundle patch files in profile order.
* The page text follows fixed rules: plain language, no hyphens or dashes, no semicolons. Exact npm package names and
  repository paths live in the citation register, not on the pages.
* An independent review checked every page for code truth, layout, legibility and cross page consistency.


## Stated safety posture

In the project's own words, DeepSeek Harness is a **developer preview** with breaking changes expected, it has **not
had a security audit**, and it should be **run contained**, in a disposable VM or container.


## Trademarks

Logos are trademarks of their owners and appear on the pages for identification only. This analysis is not affiliated
with or endorsed by any of them.


## License

The diagrams, images and text in this repository are licensed under
[Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/). See
[`LICENSE`](LICENSE). The license does not cover the third party logos shown on the pages, which remain the marks of
their owners.
