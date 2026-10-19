# Chart 0.1.0 (merged to main, deployed 2026-10-19): llm-init + Router model card

Goal: Router finds Strata by itself (`model_console` provider) and Lares gets a model card with
`supports_*`, `context_size` and a `reasoning_effort` parameter rule. Modelled on beclab/apps `strataqwen38iq4xsv3`.

## What changed against 0.0.5

- Second pod `llminit` (beclab/llm-init:v1.7.24, uid 1000, no GPU): downloads the model into
  `appCommon/huggingface`, serves Model Console (:8090, entrance `strataclient`) and publishes the model card.
- `sharedEntrances` + `options.LLMGatewaySupported: true` (what Router looks for).
- Engine pod waits for llm-init's hand-off (`/run/llm-init/model_download_finish`), links the downloaded GGUF
  files (+ mmproj) into `/data/gguf-hf` and starts the image's own `docker-entrypoint.sh` with `GGUF_DIR`.
  `REINSTALL=1` only when files / context / KV / vision changed (marker `/data/.strata-chart-state`).
- Model, context, KV, reasoning levels etc. are Olares envs (`envs:` in the manifest), not `values.yaml`.
- Entrance order unchanged for the first two (`strata`, `strata-api`), so the entrance host
  `771944791.<zone>` of the manual Router provider keeps working. `strataclient` is the new third entrance.
- Resource envelope grew by llm-init's share (lint demanded it): 2200m/13 CPU, 57Gi/86Gi RAM.

## Verified locally (Mac, Docker)

- llm-init v1.7.24 starts with ENGINE_KIND=strata, writes `model-spec.json`:
  `supports_*`, `parameter_rules: reasoning_effort low/medium/xhigh default xhigh`, `context_size 262144`.
- `olares-cli chart lint` OK, `helm template` renders, wrapper scripts pass `sh -n` and a stub run
  (links, ENGINE_ARGS parsing, REINSTALL logic).

## NOT verified (needs the real Olares One)

- That llm-init's proxy reaches the engine with an allowed Host (`STRATA_ALLOWED_HOSTS` now has `strata`).
- That `setup.py --gguf-dir` accepts the symlinked HF files and mmproj, and the rerun re-uses packs/MTP.
- That `--include IQ3_XXS/<file>` sources download as expected (same syntax as Olares' own chart).
- That Router lists the app as a provider and shows the model card in Lares.
- Engine still runs as root (the image writes to /opt/strata); only llm-init runs as uid 1000.

## Cost / side effects of the upgrade

- Model is downloaded again (~76 GB) into appCommon; the old copy in `appCache/strata/models` (72 GB) stays and
  can be deleted afterwards. Disk is fine (3.6 TB free).
- Upgrade restarts the engine pod: Oli (default-chat -> Strata) is offline during that time.
- Router model name will differ from the manual provider's `strata/qwen3.8-flash-next-iq3_xxs`.

## Switch plan (with the other session)

1. Upload + upgrade chart 0.1.0 (version in OlaresManifest and Chart.yaml must be bumped on each re-upload).
2. Wait until Router lists the new provider (can take > 15 min); check the card in Lares.
3. `olares-cli router route pin default-chat <new model name>`; test Oli.
4. Only then remove the manual provider `strata` (a pin without a running model gives 404 and Oli is offline).

Rollback: pin default-chat back to Qwen 27B, stop Strata, resume Qwen; `git checkout main` has the 0.0.5 chart.
