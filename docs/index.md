# VNN-COMP platform documentation

This directory documents the current VNN-COMP plugin for the shared evaluation
platform. It replaces the pre-unification wiki and deliberately omits credentials,
private access instructions, and retired implementation details.

## Start here

- [Architecture and pipelines](architecture-and-pipelines.md)
- [Toolkit submissions](toolkit-submissions.md)
- [Benchmark generation](benchmark-generation.md)
- [Scoring and counterexamples](scoring-and-counterexamples.md)
- [Operations](operations.md)

## Historical and migration

- [Migration from the legacy system](history/migration-from-legacy.md)
- [VNN-COMP 2025 final-evaluation seed](history/2025-final-evaluation-seed.md)

Participant-facing toolkit and benchmark instructions are rendered inside the application
from `vnn_comp/guides.py`; the documents here describe the implementation in more detail.

Shared accounts, roles, task state, scheduling, callbacks, compute backends, runtime
settings, security, and backup boundaries are documented by the
[core evaluation platform](https://github.com/TUMcps/core-eval-platform/tree/main/docs).

## Scope

This repository owns VNNLIB versions, evaluation modes, benchmark generation, official
counterexample validation, export layout, and VNN-specific step graphs. Shared platform
behavior is documented in core.
