# CKarthikeshwar.github.io

**Live: [ckarthikeshwar.github.io](https://ckarthikeshwar.github.io)**

Personal site of Chimirela Karthikeshwar — a Mathematics & Computing student at IISc Bengaluru. Built with [Astro](https://astro.build) and [Tailwind CSS](https://tailwindcss.com), deployed to GitHub Pages.

The homepage runs on a custom "Glass Terminal" theme: a terminal boot sequence, glass/blur panels, and a mouse-reactive particle constellation background, with each section framed as a shell command (`$ ls experience/`, `$ ls projects/`, ...).

## Structure

```
src/
├── pages/
│   ├── index.astro     # Homepage — About / Experience / Projects
│   ├── dump/           # Writing — post list and individual posts
│   ├── projects/       # Standalone project pages (e.g. CoinFlow)
│   └── v1.astro        # Frozen pre-redesign homepage snapshot
├── components/          # Astro components, incl. shared home/ pieces
├── layouts/             # Page layouts (Home, Dump)
├── styles/              # Global styles + the Glass Terminal theme
└── content/blog/        # Markdown posts for /dump
```

Deployment is handled by `.github/workflows/deploy.yml`, which builds and publishes to GitHub Pages on every push to `main`.

## Archive

The homepage as it looked before the "Glass Terminal" (V2) redesign is frozen at
[`/v1`](https://ckarthikeshwar.github.io/v1) — a self-contained snapshot, not
linked from the live site's navigation.

## License

The code is MIT licensed (see [LICENSE](LICENSE)). This doesn't extend to the
personal content — resume/experience details, project write-ups, blog posts,
and photos — which remain not licensed for reuse.
