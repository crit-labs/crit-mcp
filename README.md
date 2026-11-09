# crit for Claude Code and Codex

Make an open LLM run faster on your own GPU. With this plugin, your coding agent builds a bespoke inference engine
for one model on one NVIDIA GPU: it profiles the hardware, tunes CUDA kernels, tries quantization, speculative
decoding and KV-cache techniques, measures every candidate's speed and quality with crit Eval, and serves the best
one through an OpenAI-compatible API on localhost. Every engine crit builds is specialized to your model, your GPU
and your workload, instead of running a general engine as it ships.

The plugin is a thin package around [crit](https://github.com/crit-labs/crit): a skill with crit's engine-building
workflow, which the agent follows through crit's command line. In Codex it also starts `crit mcp`, crit's MCP server,
whose tools run the same commands; in Claude Code you can add that server yourself (below). The server's code lives
in crit's [mcp/](https://github.com/crit-labs/crit/tree/main/mcp) folder.

## Install

1. Install crit 0.17.2 or later where your engines run (an NVIDIA GPU with a CUDA toolkit, Python 3.11+):
   `python -m pip install "crit>=0.17.2"`. On Windows with engines in WSL, run your agent inside WSL.
2. Add the plugin:
   - **Claude Code:** `/plugin marketplace add crit-labs/crit-mcp`, then `/plugin install crit@crit-labs`
   - **Codex:** `codex plugin marketplace add crit-labs/crit-mcp`, then install `crit` from it
3. Optional, in Claude Code: add crit's MCP tools with
   `claude mcp add crit -- crit mcp --without crit_run_cost`. The skill works without them, through crit's
   command line.

## Try it

- "Build me a fast engine for Qwen3.8-27B on this GPU, and serve the best candidate."
- "Find the inference techniques for a hybrid model's KV cache and show me how candidates use them."
- "Install the crit-hub engine OWNER/NAME, serve it, and check that it is healthy."
- "Run Eval on candidate-0003 and tell me which gates it fails and why."

## What it runs and reaches

The agent runs crit's commands on your machine: in Claude Code through its shell, in Codex through
`crit mcp --without crit_run_cost`, which the plugin starts there. crit's commands build and run native code
(candidates' builds, `crit serve`, crit Eval's kernels and probes) and keep their records under the run root. crit
mcp's read-only tools are marked so, and destructive ones (stopping a run or job, sealing or selecting a candidate,
recovering a lease) are marked for confirmation.

| Where | When |
| --- | --- |
| The local filesystem | Every command: run records, logs, checkpoints and the folders a call names |
| `127.0.0.1` | `crit serve` and the tools that measure or ask a local crit serve |
| huggingface.co | `crit_pull` downloads an engine's checkpoint or prebuilt artifact; `crit_eval_fetch`, and Eval runs when the data is missing, download crit Eval's reference package |
| crit-hub.dev and the engine's git repository | `crit_pull` reads crit-hub's index and the engine's card, and fetches its source at the card's commit, which it builds and runs locally |

## Privacy

crit collects no data and sends nothing to crit-labs: no telemetry, analytics or accounts. The downloads above
send the requests any download does (your address and the file asked for) to those hosts, under their own privacy
policies. Everything the commands write stays in folders on your machine, for as long as you keep it. The plugin
leaves out `crit_run_cost`, the one MCP tool that reads an agent's transcript files, and the skill never runs
`crit run ... cost`. The full terms are in
[crit's MCP README](https://github.com/crit-labs/crit/blob/main/mcp/README.md#privacy).

## Support

Report problems in [crit's issues](https://github.com/crit-labs/crit/issues). Licensed under Apache-2.0.
