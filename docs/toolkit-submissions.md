# Toolkit submissions

A toolkit submission is a Git repository plus a revision, script directory, worker image,
benchmark selection, VNNLIB version, and execution options. Start from the
[example toolkit](https://github.com/VNN-COMP/example_toolkit).

## Required scripts

The selected script directory must provide:

- `install_tool.sh`
- `prepare_instance.sh`
- `run_instance.sh`

The worker calls:

```text
install_tool.sh v1
prepare_instance.sh v1 <benchmark> <onnx-path> <vnnlib-path>
run_instance.sh     v1 <benchmark> <onnx-path> <vnnlib-path> <result-file> <timeout>
```

For VNN-COMP the category argument is the benchmark name. Preparation is capped at 600
seconds. A non-zero preparation exit records the failure and skips the rest of that
benchmark. The run is capped by the instance timeout from `instances.csv`, defaulting to
600 seconds when absent.

`run_instance.sh` writes its verdict on the first line of the result file. When the verdict
is `sat` or `violated`, the rest of that file is retained as the counterexample passed to
the official validator.

## Docker backend status

The current VNN `local_docker` handlers mark installation and post-installation complete
without running them, and the benchmark wrapper treats private worker addresses as
backend-local. The remote-Docker service also returns a private container address that can
take the backend-local branch. Consequently Docker execution does not consistently follow
the isolated worker flow shown above.

## Evaluation choices

- VNNLIB version: `1.0` or `2.0`.
- Evaluation mode: all instances, ten random instances, or the first instance.
- Benchmark selection: selected published benchmarks, or all published benchmarks when no
  explicit selection is stored.
- Root execution: separate options exist for installation, post-installation, and toolkit
  execution. Prefer unprivileged execution.
- Pauses: a submission can pause before post-installation and after post-installation.
- Export: when enabled, each benchmark gets an export step after validation.

The shared form currently also displays a restart-after-post-installation option. The VNN
step builder stores the value but does not create a restart step, so the option has no
effect on execution.

## Post-installation

When the backend is not `local_docker`, the post-install handler runs the text submitted
through the form on the assigned worker. It is used for machine-dependent setup such as
license activation. Do not commit license keys, tokens, or credentials to the toolkit
repository, and do not print them to task logs.

A pause holds indefinitely until the owner or an administrator resumes the task. Pauses are
for interactive inspection; they do not by themselves create remote access or expose a
worker to the public network.

## Result rows

The node writes rows in this order:

```text
benchmark,onnx,vnnlib,prepare_time,result,runtime
```

The plugin associates each row with an instance named from both the ONNX and VNNLIB file
stems. Paths retain their benchmark-relative subdirectories so the scorer can resolve the
same files used during execution.
