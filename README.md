# static-website-template

A Hereya source template for static websites. Astro 5 + Tailwind CSS v4, deployed to S3 + CloudFront via the `hereya/cloudfront-deploy` package.

## Use this template

Don't clone directly — scaffold a new project with the Hereya CLI. There are two paths:

### Local

Develop on your own machine:

```bash
hereya init hereya/<your-project> \
  -w hereya/<your-workspace> \
  -d hereya/<your-staging-workspace> \
  -t hereya/github-private-repo \
  -p "sourceTemplate=hereya/static-website-template"
```

This creates a private GitHub repo from the template and clones it into the current directory.

### Devenv (remote VM)

Develop on a Hereya dev VM:

```bash
hereya devenv project init hereya/<your-project> \
  -w hereya/<your-devenv> \
  -d hereya/<your-staging-workspace> \
  -t hereya/github-private-repo \
  -p "sourceTemplate=hereya/static-website-template"
```

This creates the GitHub repo and clones it onto the devenv worker. Connect with `hereya devenv ssh -w hereya/<your-devenv>` to develop.

## Stack

| | |
|---|---|
| Framework | [Astro](https://astro.build) 5 (`output: 'static'`) |
| Styling | [Tailwind CSS](https://tailwindcss.com) v4 (via `@tailwindcss/vite`) |
| Build output | `dist/` |
| Dev port | `4321` |
| Deploy target | S3 + CloudFront (via `hereya/cloudfront-deploy`) |

## Develop

```bash
hereya up                          # provision Hereya packages (no-op for this template)
npm install
hereya run -- npm run dev          # binds to http://localhost:4321
```

If you're on a devenv, expose the dev server to your local browser via SSH `-L` or a cloudflared quick-tunnel:

```bash
cloudflared tunnel --url http://localhost:4321
```

`astro.config.mjs` allowlists `*.trycloudflare.com` hosts so the tunnel works without extra config.

## Deploy

`hereya.yaml` includes a `preDeployCommand` that runs `npm install && npm run build` on the remote executor (which clones a fresh copy of your repo on every run, so it has no `node_modules` and no `dist/`). You don't need to build manually.

```bash
hereya deploy -w hereya/<your-staging-workspace>
```

The deploy command provisions the S3 bucket + CloudFront distribution and uploads `dist/`. The CloudFront URL is printed at the end (`DistributionDomainName`).

To tear it down:

```bash
hereya undeploy -w hereya/<your-staging-workspace>
```

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
```

## Customize

- **Theme tokens** — Tailwind v4 reads custom colors and fonts from a `@theme { ... }` block in `src/styles/global.css`. No `tailwind.config.js` needed.
- **Pages** — drop more `.astro` files into `src/pages/`; each becomes a route.
- **Layouts/components** — add to `src/layouts/` and `src/components/`.

## License

Use as a starting point for your own projects. No warranty.
