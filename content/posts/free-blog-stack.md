---
title: "This Blog Costs $0 to Run — Here's the Exact Setup"
date: 2026-09-09T18:00:00+08:00
draft: false
description: "The full architecture behind this site: Hugo + GitHub Pages + Pages CMS, how the pieces fit together, the two pitfalls I hit while setting it up, and what it can't do."
cover: /images/free-stack-cover.jpg
tags:
  - blogging
  - hugo
  - tutorial
  - digital nomad
---

In [the previous post](/posts/hello-world/) I promised practical notes on nomadic infrastructure. Let's start with the most immediate piece of infrastructure a writer needs: a home on the internet that costs nothing, can't be defaced by hackers, and survives the death of any single company.

That's this website. It runs on three free services and a GitHub repository, and publishing a post is as easy as writing an email. Here's the whole thing.

## The stack

```
┌─────────────┐     commit      ┌───────────────┐     build     ┌──────────────┐
│  Pages CMS  │ ──────────────▶ │  GitHub repo  │ ────────────▶ │ GitHub Pages │
│  (writing)  │                 │   (source)    │  (Actions CI) │   (hosting)  │
└─────────────┘                 └───────────────┘               └──────────────┘
```

- **[Hugo](https://gohugo.io/)** — a static site generator. It turns Markdown files into HTML, thousands of pages per second. No database, no PHP, nothing to hack. The [Dream](https://github.com/g1eny0ung/hugo-theme-dream) theme adds a masonry card layout, dark mode, and built-in search.
- **[GitHub Pages](https://pages.github.com/)** — free hosting for public repositories. Your site lives at `username.github.io` with HTTPS included.
- **GitHub Actions** — free CI. On every push, it installs Hugo, builds the site, and publishes the output. I never touch this; it just works.
- **[Pages CMS](https://pagescms.org/)** — the visual admin panel. A web editor that logs into your GitHub account and commits posts on your behalf, so day-to-day writing needs zero command line. It's open source and can even be self-hosted.

The key property of this design: **your content is just files in a Git repository.** The editor, the generator, and the host are all replaceable. If Pages CMS disappeared tomorrow, I'd switch to any other Git-based CMS and keep writing. Compare that to WordPress, where your posts live in a MySQL database tied to a server you must patch, back up, and pay for.

## How it was built

The whole setup took an afternoon, most of which was writing content, not configuration:

1. **Create a repository** named `username.github.io` — GitHub serves it as a personal site automatically.
2. **Scaffold Hugo** (`hugo new site`), add the Dream theme as a git submodule, and fill in one config file: title, author, theme options.
3. **Add a GitHub Actions workflow** that installs Hugo, builds with `hugo --minify`, and deploys `public/` via the official Pages actions. [The exact file is in this site's repo.](https://github.com/bjachlxam/bjachlxam.github.io/blob/main/.github/workflows/hugo.yml)
4. **Add `.pages.yml`** — a small YAML file at the repo root that tells Pages CMS what a blog post looks like (title, date, draft flag, summary, tags, Markdown body). That's what powers the "Add an entry" button in the admin panel.

From then on, publishing = log in to Pages CMS, write, click save. The CMS commits, Actions builds, Pages deploys. About one minute from "publish" to "live".

## Two pitfalls worth knowing

These cost me some debugging, so you get them for free:

**1. GitHub hijacks the deployment mode.** For `username.github.io` repositories, GitHub auto-enables its *legacy* deployment (serve the raw repository files) the moment you push. That silently overrides the workflow-based deployment, and the site 404s or shows raw Markdown. The fix: in the repo's **Settings → Pages**, switch the source from *"Deploy from a branch"* to *"GitHub Actions"*, then re-run the workflow.

**2. The CMS config filename is `.pages.yml`.** Several tutorials floating around (including the one that inspired this setup — a post by Chinese blogger William Long) call it `.pages.config.yml`. That file is ignored, and the CMS greets you with *"configuration file does not exist yet"*. Also note the current schema uses `input`/`output` for the media directory, not `path`, and `order: desc` for list sorting.

## The honest fine print

No setup is free of trade-offs, and a blog about honest infrastructure should list them:

- **GitHub Pages limits**: site must stay under 1 GB, soft limit of ~100 GB bandwidth/month, and builds are for public repos only. Plenty for text; a photo-heavy blog should compress images or use an image host.
- **Publishing isn't instant**: ~1 minute of build time per post. Irrelevant for humans, but it's not a database-backed CMS.
- **Writing is Markdown**: the CMS editor is comfortable, but if you want drag-and-drop page layouts (Elementor-style), this isn't that.
- **Comments and analytics** are DIY: wire up something like giscus (GitHub Discussions as a comment section) or go comment-free, like this blog does for now.

## What it adds up to

Total monthly cost: **$0**. Total attack surface: one static file server. Total lock-in: none. For a blog about building a location-independent life, a website that runs itself, owns its content in plain files, and bills nothing feels like the right first brick.

The full source of this site — configuration, CMS setup, and workflow — is public at [bjachlxam/bjachlxam.github.io](https://github.com/bjachlxam/bjachlxam.github.io). Clone it, rename it, and the same stack is yours in an afternoon.
