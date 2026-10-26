# SeinARTS Documentation

The public SeinARTS documentation site is built with Astro Starlight and published at [docs.seinarts.gg](https://docs.seinarts.gg).

## Local development

Use Node.js 22.19 or newer. From the repository root:

```powershell
npm install
npm run dev
```

Create public pages under `src/content/docs/`. Update navigation and site metadata in `astro.config.mjs`, and keep visual changes in `src/styles/seinarts.css`. Guide screenshots live in `src/assets/guides/<page>/`. The logos in `src/assets/brand/` are copies of the framework's brand kit; replace them there when the brand changes.

Run the production checks before publishing:

```powershell
npm run check
npm run build
```

## Publishing

The workflow at `.github/workflows/deploy-docs.yml` builds the site and deploys it to GitHub Pages whenever a commit reaches `main`.

The GitHub repository must use **GitHub Actions** as its Pages source and set **docs.seinarts.gg** as its custom domain. DNS should keep `docs.seinarts.gg` as a CNAME to `rjphenom.github.io`.
