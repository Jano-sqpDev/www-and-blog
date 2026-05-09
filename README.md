# SQP Dev — Website & Blog

Built with [Hugo](https://gohugo.io/), hosted on [Cloudflare Pages](https://pages.cloudflare.com/).

## Local development

```bash
hugo server -D
```

Visit `http://localhost:1313`

## New blog post

```bash
hugo new posts/my-post-title.md
```

Edit the file in `content/posts/`, then commit and push. Cloudflare Pages will auto-deploy.

## Cloudflare Pages settings

- **Build command:** `hugo`
- **Build output directory:** `public`
- **Environment variable:** `HUGO_VERSION` = `0.147.0`
