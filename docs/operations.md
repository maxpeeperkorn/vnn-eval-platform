# Operations

Shared deployment, security, scheduler, administration, and backup guidance lives in the
[core documentation](https://github.com/TUMcps/core-eval-platform/tree/main/docs). This
page contains VNN-specific configuration and checks.

## Local development

```bash
git clone --recurse-submodules https://github.com/VNN-COMP/vnn-eval-platform.git
cd vnn-eval-platform
docker compose up --build
```

The frontend is available at `http://localhost:5173` and the backend at
`http://localhost:8000`. The first signup becomes the enabled admin. The Compose stack uses
PostgreSQL, the pinned core frontend, persistent data storage, and `local_docker` workers.

The Docker worker lifecycle is implemented, but the VNN install, generation, run, scoring,
and export scripts still contain a mixture of backend-local and worker-local path handling,
and the test suite does not exercise a complete real Docker submission.

More importantly, private worker addresses are currently classified as backend-local by
several VNN wrappers. This includes normal local-Docker addresses and can include addresses
returned by the remote-Docker service. With `local_docker`, generator and toolkit run
helpers use backend-container paths, while toolkit installation and post-installation are
marked complete without executing. The backend container also has database configuration
and the host Docker socket, so these backend-local branches do not provide an isolated
execution boundary for submitted code.

Development uses a fixed example `BENCHMARK_SEED`. Competition deployments supply their
season-specific seed through the same setting.

## VNN-specific production settings

Review at least:

- `COMPETITION_YEAR`
- `BENCHMARK_SEED`
- `SCORING_REPO` and `SCORING_REF`
- `BENCHMARKS_PUSH_REPO` and its deploy-key path, if exporting remotely
- `RESULTS_PUSH_REPO` and its deploy-key path, if exporting remotely

Unset push repositories use persistent local Git repositories under `DATA_DIR`. Protect and
back up that volume together with PostgreSQL.

Never put key material, access tokens, authenticated repository URLs, cloud credentials,
license keys, or production `.env` values in these documents or the repository.

## Dedicated Docker worker

The current shared remote Docker service can be run from the pinned checkout:

```bash
docker compose run --rm -p 9001:9001 backend \
  python deploy/manage.py worker_service --host 0.0.0.0 --port 9001
```

Select `remote_docker` in Admin > Settings and configure the global worker address or a
user-specific override. Restrict the worker control service to trusted network access.
This implements worker lifecycle only: the service returns a container-private address,
and current VNN wrappers can classify it as backend-local. The repository has no
end-to-end VNN task-flow coverage for this mode.

## AWS status

Core contains the EC2 adapter and VNN retains ENI/MAC submission fields. However, the
current repositories do not ship the complete AWS lifecycle shell set expected by that
adapter. AWS is therefore a deployment integration, not a turnkey mode from this checkout.
See the core compute-backend document before enabling it.

The shared Users page does not currently create or delete ENIs, and there is no manual AWS
refresh action. ENI values maintained by an external process are read from the user records.

## CI and core updates

GitHub Actions runs VNN tests, pinned core tests, frontend checks/build, Compose validation,
and a backend-image build. The old GitLab runner instructions do not apply.

Update the pinned core only after its corresponding change has merged:

```bash
core/scripts/bump-core.sh
git commit -m "chore: bump core"
```

## VNN checks after a change

1. Generate a small benchmark with a known seed.
2. Confirm the resolved commit and generated instance records are stored.
3. Run the example toolkit in a reduced evaluation mode.
4. Confirm partial results and logs appear during execution.
5. Exercise a valid and invalid/missing counterexample and inspect the frozen summary.
6. Exercise local export, and remote export only in a dedicated test repository.
7. Confirm the worker shuts down and no untracked containers remain.

## Execution boundaries

- Docker execution has no full real-worker end-to-end VNN test. Private local or remote
  container addresses may be treated as backend-local; `local_docker` also skips toolkit
  installation and post-installation.
- AWS is not turnkey without deployment-supplied lifecycle scripts and credentials.
- `vnn_comp/scripts/benchmark/export_benchmark.sh` calls the unregistered `successdocker`
  callback route, so normal benchmark exports remain active and do not auto-publish.
- A completed benchmark-export step may publish the catalog entry even when a best-effort
  export did not reach a configured remote destination.
- The track scoreboard does not currently incorporate counterexample-validation severity.
- Restart-after-post-installation is displayed but does not create a restart step.
