---
layout: post
title: Introducing GitBlog
date: "2026-02-22"
---
GitBlog is a Chrome extension that turns your browser into a full content management system for Jekyll + GitHub Pages blogs. There's no backend, no server to maintain — just your browser talking directly to GitHub's API.

If you've ever wanted a simple, visual way to manage your Jekyll blog without touching the terminal, this guide walks you through getting started.

## Prerequisites

- A **GitHub account**
- **Google Chrome** (or any Chromium-based browser)

That's it. GitBlog handles the rest — including scaffolding an entire Jekyll site from scratch if you don't have one yet.

## Installation

Install GitBlog from the Chrome Web Store. Once installed, you'll see the GitBlog icon in your browser toolbar.

## Connecting to GitHub

Click the GitBlog toolbar icon to open the popup.

### Creating a Personal Access Token

GitBlog authenticates using a **fine-grained GitHub Personal Access Token (PAT)**. To create one:

1. Go to [GitHub Settings > Developer settings > Personal access tokens > Fine-grained tokens](https://github.com/settings/tokens?type=beta)
2. Click **Generate new token**
3. Set the following permissions:
   - **Repository access:** All repositories
   - **Contents:** Read and write
   - **Administration:** Read and write (needed for creating new repos)
4. Copy the generated token

Back in the GitBlog popup, paste your token and click **Connect**. GitBlog validates the token against the GitHub API and stores it securely in Chrome's synced storage — so it's available across your devices.

### Selecting a Repository

After connecting, GitBlog shows a dropdown of your repositories. Select one and GitBlog checks whether it has a Jekyll structure (by looking for `_config.yml`).

- **Green badge** — Jekyll detected, ready to go.
- **Yellow badge** — No Jekyll structure found. You can still open the editor, but you may need to set up Jekyll first.

Click **Open GitBlog** to launch the full editor in a new tab.

### Creating a New Blog from Scratch

Don't have a Jekyll blog yet? If you don't already have a `{username}.github.io` repository, GitBlog offers a **Create** button right in the popup. Click it and GitBlog will:

1. Create the `{username}.github.io` repository on GitHub
2. Scaffold the full Jekyll structure — layouts, includes, CSS, a default about page, and a "Hello World" first post

A progress bar shows each file being created. Once done, click **Open GitBlog** and start writing.

## Using the Editor

The editor opens in a full browser tab with three main sections accessible from the top navigation: **Posts**, **Sections**, and **Settings** (gear icon).

### Posts

The Posts view lists all your blog posts from both `_posts/` (published) and `_drafts/` (drafts). Each post shows its title, date, and a **Draft** badge if unpublished. You can search posts by title using the search box.

#### Creating and Editing Posts

Click **New Post** to open the post editor. The editor has two areas:

**Front Matter Form** — A compact form at the top with fields for:

- **Title** (required)
- **Date** (defaults to today)
- **Category** (optional)
- **Tags** (comma-separated, optional)

**Markdown Editor** — The main editing area with three view modes, toggled from the toolbar:

- **Editor** — Full-width text editor
- **Split** (default) — Side-by-side editor and live preview
- **Preview** — Full-width rendered preview

The preview renders your markdown in real time using the `marked` library with HTML sanitization, so you see exactly how headings, lists, code blocks, tables, and other formatting will look.

#### Inserting Images

Click **Insert Image** in the editor toolbar to upload an image. GitBlog uploads it to `assets/images/` in your repository and inserts the markdown image syntax at your cursor position. Images are limited to 5 MB.

#### Publishing and Drafts

Two save options are available:

- **Save Draft** — Saves the post to the `_drafts/` directory (not published)
- **Publish** — Saves to `_posts/` with an auto-generated filename (`YYYY-MM-DD-title-slug.md`)

You can move a post between draft and published states by using the corresponding button on subsequent saves.

### Sections (Pages)

The Sections view manages your static pages — things like "About", "Contact", or "Projects" that appear in your site's navigation.

Each section shows its title, permalink, and whether it appears in the navigation (**In nav** green badge or **Hidden** grey badge).

#### Creating Sections

Click **Add Section** to open the section editor with fields for:

- **Title** (required)
- **Permalink** (auto-generated from title if left blank, e.g., `/about/`)
- **Layout** (`page` or `default`)

Plus the same markdown editor as posts.

#### Reordering Navigation

Drag sections using the grab handle to reorder them. GitBlog updates the `header_pages` array in your `_config.yml` automatically, so your site's navigation reflects the new order immediately on the next build.

### Settings

Click the gear icon to access site settings.

**Site Settings:**

- **Blog Title** — Your site's `<title>` and header
- **Description** — The site description / tagline
- **Permalink Format** — How post URLs are structured:
  - `/:title/`
  - `/:year/:month/:title/`
  - `/blog/:title/`

All changes are written directly to `_config.yml`. Click **Save** and the status indicator confirms the update.

**Appearance:**

- **Color Mode** — Switch between Light, Dark, or System (follows your OS preference). This applies to the GitBlog editor itself and is saved across sessions.

## Tips and Good to Know

### Conflict Resolution

If someone (or you on another device) edits the same file while you have it open, GitBlog detects the conflict when you try to save. A dialog gives you two options:

- **Overwrite** — Push your version, replacing the remote changes
- **Reload** — Discard your local edits and load the latest version from GitHub

### Rate Limits

GitBlog displays your GitHub API rate limit in the status bar at the bottom of the editor. GitHub allows 5,000 requests per hour for authenticated users. Under normal usage you won't come close, but it's there if you need to check.

### Caching

File listings are cached locally for 5 minutes to keep the editor snappy and reduce API calls. The cache is automatically invalidated whenever you create, update, or delete a file — so you always see fresh data after making changes.

### Online / Offline Indicator

The status bar shows your connection state. If you lose connectivity, GitBlog lets you know so you don't lose work trying to save to an unreachable API.

### Synced Across Devices

Your token, selected repository, and color mode preference are stored in Chrome's synced storage. Open Chrome on another computer and GitBlog is ready to go with the same configuration.

## Summary

GitBlog gives you a clean, focused writing environment for Jekyll blogs without leaving your browser. No terminal, no git commands, no build steps — just open the extension, pick your repo, and write.
