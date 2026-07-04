# cursor-test

A Vue 3 + TypeScript + Vite project.

## Setup

```bash
npm install
```

## Development

```bash
npm run dev
```

## Build

```bash
npm run build
```

## Preview

```bash
npm run preview
```

## Deployment

Every push to `develop` builds the app and publishes it to the `gh-pages` branch via [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml).

### One-time setup

1. Go to **Settings → Pages** in the repo
2. Set **Source** to **Deploy from a branch**
3. Choose branch `gh-pages` and folder `/(root)`
4. Save and wait about a minute

### Live URL

```
https://nikomax.github.io/cursor-test/
```

**Important:** `https://nikomax.github.io/` is a different project (Films). This app is only available at the `/cursor-test/` path above.

This template uses Vue 3 `<script setup>` SFCs. Learn more about [Vue 3](https://vuejs.org/) and [TypeScript setup](https://vuejs.org/guide/typescript/overview.html#project-setup).
