# static-website-template

A Hereya source template for static websites. Astro 5 + Tailwind CSS v4, deployed to S3 + CloudFront via the `hereya/cloudfront-deploy` package.

## Use this template

Don't clone directly — scaffold a new project with the Hereya CLI. From your local machine:

```bash
hereya devenv project init hereya/<your-project> \
  -w hereya/<your-devenv> \
  -d hereya/<your-staging-workspace> \
  -t hereya/github-private-repo \
  -p "sourceTemplate=hereya/static-website-template"
```

This creates a private GitHub repo from the template and clones it onto your dev VM.

## Stack

| | |
|---|---|
| Framework | [Astro](https://astro.build) 5 (`output: 'static'`) |
| Styling | [Tailwind CSS](https://tailwindcss.com) v4 (via `@tailwindcss/vite`) |
| Build output | `dist/` |
| Dev port | `4321` |
| Deploy target | S3 + CloudFront (via `hereya/cloudfront-deploy`) |

## Develop

On the dev VM:

```bash
hereya up                          # provision Hereya packages (no-op for this template)
npm install
hereya run -- npm run dev          # binds to http://localhost:4321
```

Preview from your local machine — either an SSH `-L` tunnel or a `cloudflared` quick-tunnel works:

```bash
# Cloudflared (anonymous, ephemeral URL — fine for sharing during dev)
cloudflared tunnel --url http://localhost:4321
```

`astro.config.mjs` allows `*.trycloudflare.com` hosts so the tunnel works without extra config.

## Deploy

`hereya.yaml` includes a `preDeployCommand` that runs `npm install && npm run build` on the remote executor (which clones a fresh copy of your repo on every run, so it has no `node_modules` and no `dist/`). You don't need to build manually before deploying.

```bash
hereya deploy -w hereya/<your-staging-workspace>
```

The deploy command provisions the S3 bucket + CloudFront distribution and uploads `dist/`. The CloudFront URL is printed at the end (`DistributionDomainName`).

To tear it down:

```bash
hereya undeploy -w hereya/<your-staging-workspace>
```

### Optional: CI/CD with AWS CodeBuild

`buildspec.yml` is provided for users who want push-to-main to trigger deploys via CodeBuild. To enable it, create a CodeBuild project that watches your repo and set these environment variables:

- `HEREYA_TOKEN` — your Hereya API token
- `HEREYA_CLOUD_URL` — your Hereya cloud endpoint
- `DEPLOY_WORKSPACE` — e.g. `hereya/<your-staging-workspace>`

Without this wiring, the buildspec is dormant and you deploy manually as above.

## Repo layout

```
src/
  layouts/Base.astro     # HTML shell
  pages/index.astro      # homepage
  styles/global.css      # Tailwind import + @theme tokens go here
public/
  favicon.svg
astro.config.mjs         # Vite + Tailwind plugin, allowedHosts, output: static
hereya.yaml              # deploy package + preDeployCommand
buildspec.yml            # optional CodeBuild config
```

## Customize

- **Theme tokens** — Tailwind v4 reads custom colors and fonts from a `@theme { ... }` block in `src/styles/global.css`. No `tailwind.config.js` needed.
- **Pages** — drop more `.astro` files into `src/pages/`; each becomes a route.
- **Layouts/components** — add to `src/layouts/` and `src/components/`.

## License

Use as a starting point for your own projects. No warranty.
