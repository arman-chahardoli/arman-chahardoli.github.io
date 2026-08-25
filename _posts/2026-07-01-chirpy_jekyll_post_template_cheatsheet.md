---
title: "Building a Personal Linux Knowledge Base"
date: 2026-07-07 20:30:00 +0330
categories: [Linux, Knowledge-Base]
tags:
  - linux
  - emacs
  - orgmode
  - github
  - documentation
  - devops
author: arman
description: >
  A complete walkthrough of creating a personal Linux
  documentation website using Chirpy, Jekyll,
  GitHub Pages, Org Mode and Emacs.
image:
  path: cover.webp
  alt: Screenshot of the final knowledge base homepage.
  lqip: cover-lqip.webp
media_subpath: /assets/posts/linux-kb
toc: true
comments: false
math: false
mermaid: true
pin: false
published: false
---
# Introduction
Welcome to my Linux knowledge base.
> This article is updated regularly.
{: .prompt-info }
> Always keep backups before changing your system.
{: .prompt-warning }
> Test everything inside a VM first.
{: .prompt-tip }
> Never blindly copy commands from the Internet.
{: .prompt-danger }
---
## Image
![Homepage](homepage.webp)
_The generated homepage._
---
## Floating image
![Architecture](architecture.webp){: .right w="350" h="240"}
This paragraph flows around the image.
---
## Shadow
![Terminal](terminal.webp){: .shadow }
---
## Dark / Light images
![Light](diagram-light.svg){: .light }
![Dark](diagram-dark.svg){: .dark }
---
## Code
```bash
git clone https://github.com/arthas-lich/arthas-lich.github.io
bundle exec jekyll serve
```
---
## File path
`~/.config/alacritty/alacritty.toml`{: .filepath}
---
## Inline code
Run `bundle exec jekyll serve`.
---

## Mathematics
$$
f(x)=x^2+1
$$

Inline equation:

$$x^2+y^2=z^2$$
---
## Mermaid
```mermaid
flowchart TD
    A[Write Notes]
    B[Commit]
    C[GitHub]
    D[GitHub Pages]

    A --> B
    B --> C
    C --> D
```
---
## Table

| Tool | Purpose |
|------|---------|
| Emacs | Writing |
| Git | Version control |
| Chirpy | Blog theme |
| GitHub Pages | Hosting |

---
## Task list
- [x] Install Jekyll
- [x] Configure Chirpy
- [ ] Publish first article
- [ ] Add search
---
## Quote
> Simplicity is prerequisite for reliability.
---
## Footnote
Linux is fun.[^1]
[^1]: Especially when documented well.
