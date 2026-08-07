# ops-heartbeat-monitor

External dead-man monitor for the Windows automation host (`TALYSSON`) that
runs the Leangency scheduled jobs and the self-hosted Actions runner while
private-repo GitHub-hosted minutes are exhausted (until 2026-09-01).

How it works:

- Each scheduled task on the host writes a heartbeat (timestamps, run counts,
  semantic result) to a **secret gist** after every run. The host itself beats
  every 10 minutes.
- The [Dead-man monitor](.github/workflows/monitor.yml) workflow here runs
  every 30 minutes **on GitHub's infrastructure** (public repo → free, and
  independent of the monitored machine) and fails when any heartbeat is stale
  or the last result was not a semantic SUCCESS.
- A failing scheduled run triggers GitHub's failure notification email to the
  owner. That is the alert channel — no third-party accounts involved.

This repo contains **no secrets and no business data**: the gist id lives in
the `HEARTBEAT_GIST_ID` repo secret, and heartbeat payloads are aggregate-only
(timestamps and counts).

Monitored right now:

| heartbeat | cadence | stale threshold |
|---|---|---|
| host (`host.json`) | 10 min | 30 min |
| outbox-drain (`outbox-drain.json`) | 15 min | 45 min, or lastResult ≠ SUCCESS |
