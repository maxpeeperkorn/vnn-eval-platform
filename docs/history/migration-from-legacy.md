# Migration from the legacy VNN system

The old wiki describes the pre-unification VNN application. Many domain concepts survived,
but model names, paths, APIs, settings, deployment, and some features changed.

## Feature mapping

| Legacy concept | Current status or replacement |
| --- | --- |
| VNN-specific Django project and polymorphic task classes | Replaced by clean shared core models plus VNN step handlers keyed by `kind`. |
| `_db_` fields and `save_new` conventions | Removed from the current schema; a one-off core migration command exists for legacy data. |
| `VNNCompSettings` | Replaced by shared `RuntimeSettings`. |
| AWS-only orchestration | Replaced by a compute-backend axis; local and remote Docker are implemented, while AWS needs deployment integration. |
| Hardcoded task/status APIs | Replaced by shared REST task, result, form, and callback endpoints. |
| Binary admin flag | Replaced by `user`, `organizer`, and `admin` roles. |
| Admin impersonation | Not currently available. |
| Admin ENI create/delete actions | Not currently available in the shared admin UI. ENI/MAC fields remain. |
| Manual AWS refresh button | Not currently available; the scheduler performs reconciliation. |
| Restart-after-post-installation | Still shown by the form but not implemented by the VNN step graph. |
| Sciebo/WebDAV artifact upload | Removed from the current plugin. Git/local-repository exports are used. |
| GitLab CI and tagged runner | Replaced by GitHub Actions. |
| Host-specific production layout | Not part of public repository documentation; keep it in a protected runbook. |

## Settings compatibility warning

The shared settings model retains `terminate_at_end`, `terminate_on_failure`,
`allow_non_admin_login`, and `allow_full_evaluation`, but current execution/login/form code
does not enforce those values. They are shown for migration compatibility and do not act as
operational switches. See the core administration docs for the current setting-by-setting
status.

## Material retained after rewriting

The following legacy subjects remain relevant and have been rewritten here:

- task and callback concepts, now expressed through shared core;
- toolkit script arguments and per-instance timeouts;
- benchmark generation and deterministic seeds;
- result export and counterexample validation;
- log polling and worker-reachable callbacks;
- the public 2025 final-evaluation seed record.

Old source paths, endpoint names, hostnames, user credentials, keys, environment contents,
and deployment commands were not transferred. Any credential found in legacy documentation
must be considered exposed and rotated rather than copied.
