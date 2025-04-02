# 🌸 Shuchang Wen's Blog · Powered by Astro + Fuwari

Welcome to my personal blog site! This website is built using the [Fuwari](https://github.com/saicaca/fuwari) template and [Astro](https://astro.build), and is deployed on [GitHub Pages](https://pages.github.com/).

👉 Live Site: [https://nanshanvv.github.io/shuchangwen-webpage](https://nanshanvv.github.io/shuchangwen-webpage)

---

## ✨ Features

- 🚀 Built with [Astro](https://astro.build/) & [Tailwind CSS](https://tailwindcss.com/)
- 💡 Light / Dark mode
- 🌈 Custom theme colors & banner
- 📱 Responsive design
- 💬 Comment system
- 🔍 Search & Table of Contents
- 🌐 Supports multiple languages (中文 / English / 日本語 / etc.)

---

## 📦 Local Development

### 1. Install dependencies

```bash
pnpm install
pnpm add sharp
```

### 2. Start development server

```bash
pnpm dev
```

Visit `http://localhost:4321` in your browser to view the site locally.

---

## 📝 Create a New Blog Post

```bash
pnpm new-post your-post-name
```

This will create a new file at `src/content/posts/your-post-name.md`.  
Edit the Markdown file and update the frontmatter:

```md
---
title: "My First Post"
published: 2025-04-02
description: "This is an example post."
tags: [Blog, Astro]
category: Learning
draft: false
lang: zh
---
```

---

## 🚀 Deployment (via GitHub Actions)

This site is automatically deployed to GitHub Pages using [withastro/action](https://github.com/withastro/action).

### Astro Configuration

In `astro.config.mjs`, set:

```js
export default defineConfig({
  site: 'https://nanshanvv.github.io/shuchangwen-webpage',
  base: '/shuchangwen-webpage/',
});
```

### GitHub Pages Settings

- Source: **GitHub Actions**
- Make sure the `pnpm-lock.yaml` file is committed to the repository
- Deploy workflow is defined in `.github/workflows/deploy.yml`

---

## 📚 License

This project is based on the [Fuwari Template by @saicaca](https://github.com/saicaca/fuwari), released under the MIT license.
