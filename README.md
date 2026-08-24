# fieldsync-updates — OTA host seed

This folder seeds the **`irfansipai/fieldsync-updates`** GitHub repo, which hosts
the Capacitor app's over-the-air update manifest + bundles via GitHub Pages.
The FieldSync app fetches `https://irfansipai.github.io/fieldsync-updates/manifest.json`
on every launch (`frontend/src/lib/ota-update.ts`) and applies whatever's newer
on the next cold start. `frontend/tools/publish-ota.mjs` writes the manifest +
`bundles/*.zip` here on each release.

## One-time setup (do this once)

1. **Create the repo** on GitHub: `irfansipai/fieldsync-updates`. Make it
   **public** (GitHub Pages on a private repo needs a paid plan; the manifest +
   bundles are the public web bundle, not secrets — public is fine).
2. **Clone it as a sibling** of `fieldsync-frontend` (the publish script expects
   `../fieldsync-updates`):

   ```bash
   cd ..            # so you're beside fieldsync-frontend/
   git clone git@github.com:irfansipai/fieldsync-updates.git
   ```
3. **Copy these seed files** into the new repo (overwrite anything GitHub made):

   ```bash
   cd fieldsync-frontend
   cp -r tools/ota-host-seed/. ../fieldsync-updates/
   cp tools/ota-host-seed/.nojekyll ../fieldsync-updates/   # dotfile, cp -r above may skip it
   ```

   (`.nojekyll` must be at the repo root — it tells GitHub Pages to serve raw
   files without Jekyll, so the `.github/` workflow + `manifest.json` work.)
4. **Commit + push** from the updates repo — the `pages.yml` workflow deploys it:

   ```bash
   cd ../fieldsync-updates
   git add -A
   git commit -m "Seed OTA host"
   git push
   ```
5. **Enable Pages**: repo Settings → Pages → Build and deployment → Source =
   **GitHub Actions** (the included `pages.yml` handles deploys).

## Publishing an update (repeat this)

From the `fieldsync-frontend` repo:

```bash
# 1. Create .env.production.local with the prod backend URL (one-time):
#    echo 'NEXT_PUBLIC_API_URL=https://<your-koyeb-backend>' > .env.production.local
# 2. Ship the OTA bundle (bumps version, builds out/, zips, updates manifest):
npm run publish:ota
```

## Notes

- OTA updates the **web bundle only** (JS/HTML/CSS). Native changes (new plugins,
  permissions, appId, AndroidManifest) need a new `.apk` built by
  `fieldsync-frontend/.github/workflows/build-apk.yml`.
- The seed `manifest.json` is `{version:"0.0.0", url:"", checksum:""}` — the app
  treats an empty `url` as "up-to-date", so a freshly-seeded host never triggers
  a bogus download. The first `publish:ota` overwrites it with a real release.
