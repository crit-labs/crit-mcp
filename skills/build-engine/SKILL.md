---
name: build-engine
description: Build, measure and serve a bespoke inference engine for an open LLM on the user's own NVIDIA GPU with crit. Use when the user wants a model to run faster on their GPU, a custom or specialized inference engine, CUDA kernel or quantization work on one model, speculative decoding, a crit run, crit Eval results, or a crit-hub engine installed and served.
---

# Build an engine with crit

crit builds one engine for one model on one GPU: candidates that each try a strategy, measured with crit Eval,
recorded through `crit run`, and served through `crit serve`, an OpenAI-compatible API on localhost. You drive it
through crit's command line, or through crit's MCP tools (`crit_*`) when the `crit` MCP server is connected: they
run the same commands.

## Before you start

- crit 0.17.2 or later must be installed where the engines run (`python -m pip install "crit>=0.17.2"`), with an
  NVIDIA GPU and a CUDA toolkit. Check with `crit --version`; if it is missing, say so and give that install
  command.
- **Read the workflow first.** The installed crit carries its own rules for the release it is: read AGENTS.md at
  that release's tag, `https://github.com/crit-labs/crit/blob/vVERSION/AGENTS.md` (VERSION from `crit --version`),
  or the `crit://AGENTS.md` resource when the crit MCP server is connected. A run's brief (`agent_request.md`)
  names the same guides. Their rules win over this summary when they differ.
- **Name the folders.** Pass the crit project folder (`--project`, the folder of `settings.toml` and `runs/`) as an
  absolute path in the user's workspace on every `crit run` command, and keep every other path absolute too.
- Each command's `--help` gives its arguments; `crit run --help` lists the run's steps.

## The workflow

1. **Set up.** Agree the model, GPU, workload, quality criteria and limits with the user. Probe the device
   (`crit eval probe`), create the run (`crit run --project P create`) and study the model
   (`crit run --project P --id RUN recon`).
2. **Start from what exists.** If crit-hub has an engine for the model, start from it (`crit pull`).
3. **Build candidates.** Plan each one (`crit run ... plan`), allocate it (`crit run ... candidate`), build and
   package it, and measure it with Eval (`crit run ... eval`); `crit eval status` names the next goal. Search crit's
   techniques (`crit inference search`) and kernels (`crit kernels search`) before writing new code.
4. **Long commands.** Builds and Eval take a while: run them in the background, follow their logs, and do the next
   round's CPU work while the GPU is busy.
5. **Deliver.** Check the package (`crit run ... serve --check`), seal and select the best candidate
   (`crit run ... seal`, `crit run ... select`), serve it (`crit serve`), and stop the run with its critique
   (`crit run ... stop`).

## Rules that always hold

- Never fit the engine to Eval's suite, and never invent a measurement: label anything unmeasured.
- Never run `crit run ... cost`: it reads the agent's transcript files.
- Ask before destructive steps: stopping a run, sealing or selecting a candidate, killing a running build or Eval.
- Give the user the served candidate's `crit serve` command and the endpoint, `http://127.0.0.1:8000/v1`.
