# Architecture and pipelines

VNN-COMP is a competition plugin for the shared evaluation platform. This repository
contains the `vnn_comp` Django app, VNN-specific node scripts, generation and scoring
integration, deployment configuration, tests, and a pinned `core` submodule.

VNN-COMP uses one implicit `default` category. Its benchmarks are divided into the ordered
groups `default`, `regular`, and `extended`; organizers combine published benchmarks into
tracks for scoreboards.

## Toolkit pipeline

The current plugin builds this ordered graph:

```text
Create Submission
  -> Assign Worker
  -> Install Toolkit
  -> optional Pause
  -> Post-Installation Script
  -> optional Pause
  -> for each benchmark:
       Run Benchmark
       Validate Counterexamples
       optional Export Results
  -> Shutdown
```

The tool revision, selected benchmarks, VNNLIB version, evaluation mode, root-execution
choices, post-install content, and export choice are recorded with the durable tool entry.
Each benchmark runs as its own step so failure or operator abort can be isolated and
partial progress can be preserved.

The counterexample-validation stage is separate from execution. It runs the official
scoring repository against the result CSV and witnesses, then freezes a structured summary
on the step for the submission page. Export is present only when requested.

## Benchmark pipeline

A proposed benchmark creates this graph:

```text
Create Submission
  -> Assign Worker
  -> Set Up Generator
  -> Generate Instances
  -> Convert VNNLIB 1.0 to 2.0 (only for a 1.0 submission; best effort)
  -> Export Benchmark
  -> Shutdown
```

Generation attempts to record the resolved source commit and generated instances. The
export handler publishes the benchmark when its step completes, without a separate manual
publish step. The callback and worker-loss behavior of this step is described under
[Execution boundaries](operations.md#execution-boundaries).

## Repository roles

Three repository types participate:

| Repository | Responsibility |
| --- | --- |
| This platform repository | Orchestration, plugin handlers, parsing, validation integration, and UI metadata |
| Benchmarks repository | Generated ONNX/VNNLIB assets and `instances.csv` consumed by tool runs |
| Results/scoring repository | Official scorer and counterexample validator; optional exported run artifacts |

When no remote benchmark/results repository is configured, development exports use local
Git repositories beneath `LOCAL_REPOS_DIR`.

## Worker model

Long-running scripts write logs and call the shared success/failure endpoints. Core polls
bounded log tails and partial results while work is active. The supplied Compose stack uses
local Docker. Remote Docker implements the same worker lifecycle through a separate service.
VNN task scripts use a mixture of worker and backend paths on both Docker modes. The AWS
adapter depends on lifecycle scripts and configuration outside these repositories.
