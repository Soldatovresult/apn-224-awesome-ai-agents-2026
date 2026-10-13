# Agent Harness Research

**Checked 13 October 2026** · [Back to the index](../README.md) · [Update audit](UPDATE-2026-10-13.md)

A harness is the software around a model: its agent loop, tools, context management, permissions, recovery and verification. A capable model can behave differently when those parts change. Compare the complete system on the work you need it to finish.

## Choose the execution surface

| Harness | Useful starting point | What to inspect before adopting |
|---|---|---|
| [Codex](https://github.com/openai/codex) | Repository work with terminal tools, sandboxing and configurable approvals | Exact CLI or app surface, model route, sandbox boundary and approval mode |
| [Claude Code](https://github.com/anthropics/claude-code) | Repository workflows with hooks, skills and subagents | Loaded instructions, hooks, provider route and tool permissions |
| [OpenCode](https://github.com/anomalyco/opencode) | Coding across providers with configurable agents and tools | Resolved model, provider compatibility and ambient configuration |
| [pi](https://github.com/earendil-works/pi) | A small coding-agent core with an extensible toolkit | Extensions, system prompt, context policy and provider support |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | Terminal work in the Gemini ecosystem | Authentication path, quotas, sandbox configuration and MCP access |
| [OpenHands](https://github.com/OpenHands/OpenHands) | Development agents operating in managed execution environments | Runtime isolation, resource allocation, skills and environment parity |
| [Hermes Agent](https://github.com/NousResearch/hermes-agent) | Persistent personal workflows across tools and channels | Memory lifecycle, skills, tool scopes and background-job limits |
| [Deep Agents](https://github.com/langchain-ai/deepagents) | Building a custom harness with planning, files and delegated work | Backend storage, subagent state, tool access and interruption behavior |

This is a map of capabilities documented upstream. These products have not been run against one shared task suite for this update.

## Public comparisons worth reading

| Primary source | Design | Evidence boundary |
|---|---|---|
| [OpenBench](https://github.com/minghinmatthewlam/openbench) | Keeps model and task fixed while varying CLI harnesses; uses external checkers and Harbor-based execution | Separate benchmark families and task tiers. Its README says some Terminal-Bench results remain local-only; do not cite those as a published dataset. |
| [OpenBench M3](https://openbench.run/openbench/releases/2026-07-02-m3/) | Five harnesses, three tasks and five trials with a shared GPT configuration | The tested harnesses reached the correctness ceiling on this small task set. Efficiency differences do not establish a general quality ranking. |
| [d1-m4ss/harness-benchmark](https://github.com/d1-m4ss/harness-benchmark) | Controlled model series with pinned workspaces and requested-versus-actual model checks | An initial Claude route used a fallback model. The authors preserve it separately from the controlled comparison. |
| [AmirinNS/harness-bench](https://github.com/AmirinNS/harness-bench) | Same-model CLI comparisons with deterministic checks, configuration isolation and blinded judging | Its clean pilot has four tasks and one trial per cell. Tool-specific token meters differ; token-derived prices are estimates. |
| [rajshah4/harness-benchmark](https://github.com/rajshah4/harness-benchmark) | Publishes prompts, external verifiers, traces and provider usage ledgers | The clean AWS comparison excludes a Codex control that used another model and environment. Read each later experiment's configuration separately. |
| [Finding the Right Fit](https://arxiv.org/abs/2610.00917) | Research paper evaluating 66 model-harness configurations across three task settings, including native Codex and Claude Code pairings | The authors report ranking reversals across harnesses and tasks. This supports evaluating a pairing on a target workload; it does not establish a universal winner. |
| [Anthropic: infrastructure noise](https://www.anthropic.com/engineering/infrastructure-noise) | Examines how execution infrastructure affects coding evaluations | Resource conditions belong in the experiment record. Similar prompts alone do not make two runs comparable. |

The research was checked against these public sources on 13 October. This repository did not rerun their experiments. Results for another model, version, provider or task set should stay attached to that configuration.

## A comparison you can trust

1. Freeze the task and verifier. Give each harness a fresh copy of the same repository commit. Keep the checker and expected solution outside agent write access. Confirm the checker fails before work and passes for the known solution.
2. Verify the actual model. Record the model snapshot, provider, endpoint, reasoning settings and any fallback. A requested model label is insufficient when routing can change.
3. Isolate configuration. Record loaded skills, extensions, hooks, MCP servers and instructions. Check ambient directories as well as each tool's configured home; the AmirinNS pilot documents contamination from another tool's skills.
4. Match the environment. Keep hardware, container image, network access, dependency state, concurrency and permissions comparable. Separate cold and warm caches.
5. Bound execution. Define timeout, retry count, maximum tool steps, resource budget and spend limit before a run. Count failures and retries in the denominator.
6. Measure accepted work. Report external-checker success, elapsed time, human interventions and provider-returned usage. Preserve missing cache or cost fields as unknown. Keep measured charges separate from token-based estimates.
7. Repeat and preserve evidence. Publish sample size, per-run results and the exact revisions. A single successful run is useful acceptance evidence for that task, but gives no estimate of run-to-run variance.

## Build the smallest useful harness

Start with one bounded objective and a callable verifier. Add mechanisms only when a failure shows they are needed.

| Recurring failure | Mechanism to evaluate | Observable acceptance |
|---|---|---|
| The agent loses the goal | Short objective, constraints and completion criteria; load detailed context on demand | Work satisfies the stated criteria after context grows |
| Tool errors trigger repeated blind retries | Small tool contracts with explicit status, cause and next action | A failed call leads to a bounded recovery or a clear stop |
| A long run cannot resume safely | Durable checkpoints with file and revision references | Resume preserves completed work and avoids replaying side effects |
| Workers duplicate or overwrite work | Exclusive scope, explicit handoff and an integration owner | Independent outputs integrate without foreign-work loss |
| The agent declares success too early | External tests, browser checks where relevant, and a separate review | Acceptance is tied to the exact final artifact |
| The same task consumes excess context | Retrieve context when needed; measure compaction or caching | The same verifier passes with lower measured total effort |

Useful building blocks: [Harbor](https://github.com/harbor-framework/harbor) for evaluation environments, [LangGraph](https://github.com/langchain-ai/langgraph) for durable stateful execution, and [Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) for custom applications using Claude Code's agent capabilities.

An SDK or framework supplies components. Your application still owns the permission boundary, recovery behavior and definition of done.
