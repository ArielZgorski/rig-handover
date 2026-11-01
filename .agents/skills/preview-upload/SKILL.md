---
name: preview-upload
description: Save a rig page as one HTML file and publish or re-upload it on GitHub Pages (ADR 0003). Use for a plan's "Preview for review" and "Clean up" issues.
---

# preview-upload

Host: GitHub Pages from a separate public repo (e.g. `<owner>/rig-preview`), `index.html` + `.nojekyll`
on `main`. Never commit the preview into this repo. Ask the human before each upload.

1. Save: run rig (`pnpm start`), then `curl -sf localhost:8787/digest/<night> -o index.html`.
   Clean-up copy: run with `LANE_MARKER=` set empty so the banner reads "no marker set".
2. Upload: commit `index.html` in the preview repo and push. First time only:
   `gh repo create <name> --public --source . --push`, then
   `gh api -X POST repos/<owner>/<name>/pages -f 'source[branch]=main' -f 'source[path]=/'`.
3. Verify from outside: `curl -s https://<owner>.github.io/<name>/` returns 200 and the banner
   you expect. Only then send the link, and don't close the preview issue until the reviewer has looked.

If the deploy fails with "in progress deployment … Please cancel <sha> first": mark that
deployment inactive and delete it, then rebuild:

```sh
gh api repos/<owner>/<name>/deployments --jq '.[] | "\(.id) \(.sha)"'
gh api -X POST repos/<owner>/<name>/deployments/<id>/statuses -f state=inactive
gh api -X DELETE repos/<owner>/<name>/deployments/<id>
gh api -X POST repos/<owner>/<name>/pages/builds
```
