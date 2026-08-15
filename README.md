# Singularity microblog

A small Hugo microblog published at [singularity.sg](https://singularity.sg) with GitHub Pages.

## Write a post

Install [Hugo](https://gohugo.io/installation/) (version 0.164.0 or newer), then run:

```sh
hugo new content posts/my-note.md
hugo server -D
```

Edit the new Markdown file in `content/posts/`. When it is ready, change `draft: true` to `draft: false`, commit, and push to `master`. GitHub Actions builds and publishes the site automatically.

Site metadata and profile links live in `hugo.yaml`. The visual design is in `assets/css/main.css`.

## One-time GitHub setting

In the repository, open **Settings → Pages** and set **Source** to **GitHub Actions**. Keep `singularity.sg` configured as the custom domain.
