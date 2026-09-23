# Scoring and counterexamples

## Result and witness flow

For each instance, the worker records benchmark name, ONNX path, VNNLIB path, preparation
time, verdict, and runtime. When a toolkit reports `sat` or `violated`, its result file is
also stored as a counterexample named from the network and property stems.

After the benchmark run, a separate validation step clones the configured official scoring
repository at `SCORING_REF` and processes the result CSV and counterexamples. Keeping this
stage separate makes validator failures distinguishable from toolkit execution failures.

## Configuration

The VNN deployment defines:

- `COMPETITION_YEAR` — used in benchmark and export paths;
- `SCORING_REPO` — official scorer repository used for validation;
- `SCORING_REF` — revision of the scorer;
- `RESULTS_PUSH_REPO` — optional destination for exported artifacts;
- `RESULTS_DEPLOY_KEY` — host-side path used only when a remote destination is set.

The validation repository is the trusted official scorer, not necessarily the repository to
which this deployment exports results.

## Validation summary

The plugin parses the scorer's final text summary into:

- total instances;
- verdict counts: `holds`, `violated`, `timeout`, `error`, and `unknown`;
- witness counts: valid, valid with tolerance, invalid, and missing.

Stored per-instance results are reconciled with the scorer summary because the scorer may
omit all-unknown or all-timeout categories. Invalid or missing witnesses, and explicit
errors, give the validation summary an error severity. Timeouts alone do not.

If validation produces no recognizable summary, the plugin leaves the structured summary
unset and the UI falls back to the raw result rows. Validation and export are currently
best-effort when a worker disappears during those stages.

## Export layout

When result export is enabled, artifacts are written as:

```text
<safe-tool-name>/<competition-year>_<benchmark>/
  results.csv
  *.counterexample.gz
  *.counterexample.check.json
```

The year-bearing directory is part of the scorer contract. The task details page can offer
that exported directory as an archive after a successful export step.

## Current scoreboard behavior

Organizer-managed tracks drive the platform scoreboard. The current plugin groups results
by tool and reports solved count plus total runtime, both for the whole track and for the
`default`, `regular`, and `extended` groups. A stored result other than `unknown`, `error`, or
`timeout` is counted as solved.

The current track scorer does not fold the counterexample-validation summary into its
solved count. Validation is visible on submission details, while the aggregate reflects
only stored verdicts and runtime. Witness-validity rules and penalties are not represented
in this scoreboard.

For the participant file format, follow the current season's published VNN-COMP rules and
VNNLIB specification linked from the in-application toolkit guide.
