# Benchmark generation

A benchmark submission names a benchmark, source repository, revision, VNNLIB version,
generator directory, output directories, and `instances.csv` location. New submissions
are stored in the `default` group; an administrator may change the group later.
Start from the [example benchmark](https://github.com/VNN-COMP/example_benchmark).

## Generator contract

The generator directory must contain:

```text
generate_properties.py
```

The platform invokes it as:

```text
python generate_properties.py <BENCHMARK_SEED>
```

If `requirements.txt` exists in the generator directory, the setup step creates an isolated
environment and installs it. Otherwise the current handler installs a pinned legacy
baseline environment for compatibility.

The generated layout must contain the configured ONNX directory, VNNLIB directory, and CSV
file. The normal defaults are:

```text
onnx/
vnnlib/
instances.csv
```

Each CSV row contains the two paths and may include a timeout:

```text
<onnx-path>,<vnnlib-path>[,<timeout-seconds>]
```

During toolkit execution, a missing or blank timeout defaults to 600 seconds.

The normalization step makes a best-effort rewrite when a referenced basename exists in
the declared output directory. Generation checks that the CSV path exists and that the
VNNLIB output contains at least one `.vnnlib` file. It does not validate every row, require
a non-empty CSV, or verify every referenced ONNX/VNNLIB file, so authors and operators must
perform those checks before treating the generated benchmark as usable.

## Reproducibility

Production must set `BENCHMARK_SEED` according to the season's published seed process. The
generation handler attempts to store the source commit resolved on the worker. Export
writes a README containing the public source repository, commit value, generation seed,
and export time. Use an immutable submitted commit rather than relying solely on
post-generation discovery.

Do not put private clone credentials in the repository URL or generated README. Private
source access, if supported by a deployment, must use a protected host-side mechanism.

## Conversion

For a VNNLIB 1.0 submission, conversion to VNNLIB 2.0 runs as a separate best-effort step.
A conversion failure is logged but does not fail the benchmark task; the original 1.0
output can still be exported. A VNNLIB 2.0 submission skips this step.

## Export layout

With a remote benchmarks Git repository configured, export clones, updates, commits, and
pushes. Otherwise it commits to the persistent local repository beneath
`LOCAL_REPOS_DIR/benchmarks`.

VNN uses this structure:

```text
benchmarks/<benchmark>/<vnnlib-version>/
  onnx/
  vnnlib/
  instances.csv
benchmarks/<benchmark>/README.md
```

Generated ONNX and VNNLIB files are tracked through Git LFS when available. Regenerating a
benchmark replaces the directory for that version.

The export script sends its final callback to an unregistered `successdocker` route rather
than the success route, so a normal export does not advance the step or publish the
benchmark. Worker loss is treated as a valid end for this step and can publish the catalog
entry without proving that a configured remote received the files. See
[Execution boundaries](operations.md#execution-boundaries).

## Importing an existing official repository

The `bulk_import` management command imports benchmark metadata from generated benchmark
README files. From the repository root, after cloning the official benchmark repository
inside the backend:

```bash
python deploy/manage.py bulk_import --repo-path <path-to-benchmarks-directory>
```

The importer expects source repository, source commit, and generation seed metadata and
marks successfully imported benchmarks published.
