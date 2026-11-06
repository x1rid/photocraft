# PhotoCraft Web — GitHub Pages

Hosts the PhotoCraft WebAssembly build (`site/`) on GitHub Pages, and automatically
pulls in new versions.

## One-time setup

1. Create a new GitHub repo and push this folder's contents to its `main` branch.
2. In the repo: **Settings → Pages → Build and deployment → Source** = **GitHub Actions**.
   (Don't pick "Deploy from a branch" — the `deploy.yml` workflow does the publishing.)
3. That's it. The first push to `main` runs `deploy.yml` and your site goes live at
   `https://<your-username>.github.io/<repo-name>/`.

## How updates happen

- `check-update.yml` runs on a schedule (default: daily at 06:00 UTC) and on manual
  trigger (Actions tab → "Check for PhotoCraft web update" → Run workflow).
- It asks GitHub's API for the latest release of `storytold/photocraft`, looks for the
  `photocraft-web-<version>.zip` asset, and compares the version to `site/.version`.
- If there's a newer version, it downloads the zip, replaces the contents of `site/`,
  commits the change, and pushes to `main`.
- That push to `main` (touching `site/**`) automatically triggers `deploy.yml`, which
  republishes the Pages site.

So: nothing to do manually — new PhotoCraft web releases show up on your Pages site
within a day, usually sooner if you trigger the check manually.

## Notes / things you may want to tweak

- **Schedule**: edit the `cron` line in `check-update.yml`. GitHub Actions schedules can
  run late under load; a few minutes to low hours of delay is normal.
- **Repo size**: each version replaces `site/` wholesale rather than appending, but old
  versions stay in git history. If that grows the repo more than you'd like, you can
  periodically squash history or use `git gc`, but it's not required for Pages to work.
- **Project vs. user/org Pages**: if this repo is named `<username>.github.io`, the site
  serves at the domain root instead of `/<repo-name>/`. Since PhotoCraft's `index.html`
  uses relative paths, it works fine either way — no changes needed.
- **Custom domain**: add a `CNAME` file inside `site/` with your domain, and configure
  the DNS as GitHub's Pages docs describe. The updater overwrites `site/` on every
  update, so if you add a `CNAME` by hand, also add it to the `cp -r` step in
  `check-update.yml` (or just drop it back in `site/` after each run — GitHub Pages
  settings will keep re-adding it on the next successful deploy if you set the custom
  domain in the repo settings UI, which also writes CNAME automatically to the Pages
  config regardless of the source branch content).
- **GITHUB_TOKEN permissions**: `check-update.yml` needs `contents: write` to push its
  commit (already set in the workflow). No extra secrets or PATs are needed — the
  built-in `GITHUB_TOKEN` is enough both for reading GitHub's public release API and for
  pushing back to this repo.
