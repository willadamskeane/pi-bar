# willadamskeane/pi-bar — personal fork

Fork of [tianrendong/pi-bar](https://github.com/tianrendong/pi-bar) with two local patches. Everything else tracks upstream.

## Patches on `main`

1. **Progress summaries follow the session model** (`extensions/status-footer.ts`, `getFastProgressModelAuth`).
   Upstream picks its progress-summary model from a fixed fast-model list (`openai-codex/*`, `anthropic/claude-haiku-*`).
   When none of those have credentials, the progress segment stays empty. On a machine that only runs a local
   llama-swap server, pinning a different model also forces the server to unload and reload between checkpoints.
   The patch resolves auth from `ctx.model` first, so the summary rides the model already loaded, and a
   `progressModel` pin (via `pi-bar.json`, `PI_BAR_PROGRESS_MODEL`, or `/bar`) becomes a fallback only.

2. **Two-line footer under width pressure** (`layoutFooterLines` in `extensions/status-footer.ts`).
   When the footer cannot fit on one line as built and there is an active task, the task gets its own line above
   the `model ❯ thinking ❯ context` line instead of being clipped or squeezed to a stub. Badges stay on the core
   line. Fits on one line as before when everything fits.

## Install

```bash
pi remove npm:pi-bar          # if the npm package is configured
pi install git:github.com/willadamskeane/pi-bar
```

Pi clones the fork, resets it to the remote `main` on each start/`/reload`, and reconciles when the branch moves.

## Keeping up with upstream

`.github/workflows/sync-upstream.yml` runs daily (and on `workflow_dispatch`). It fetches
`tianrendong/pi-bar` `main` and rebases the fork's `main` onto it, then force-pushes with a lease.

- If the rebase is clean, the fork moves forward and the next Pi start picks the new commit up.
- If a patch conflicts with upstream, the workflow fails (GitHub notifies the repo owner) and the fork stays on
  the last good commit. Rebase locally, resolve, and push:

  ```bash
  git fetch upstream
  git rebase upstream/main
  git push --force-with-lease origin main
  ```

`main` is the fork's default branch, so scheduled workflows are allowed to run from it. Do not expect this fork's
`main` to stay a byte-for-byte mirror of upstream — it always carries these patches on top.

## Tests

```bash
npm install
npm test          # node:test via tsx
npm run check     # tsc
```
