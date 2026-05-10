---
title: "Adding HTTPS with Caddy and DuckDNS"
date: 2026-05-10
draft: false
tags:
  - caddy
  - https
  - duckdns
  - devops
categories:
  - Infrastructure
description: "Using Caddy and DuckDNS to add HTTPS to a Dockerized EC2 deployment."
---

# Adding HTTPS with Caddy and DuckDNS

After deploying the application to EC2, it initially worked only through:

```text
http://EC2-IP
```

The next step was understanding how HTTPS is normally added.

## The Stack

I used:

- DuckDNS for a free test domain
- Caddy as a reverse proxy
- Let's Encrypt certificates managed automatically by Caddy

Final setup:

```text
https://dsuopd.duckdns.org
```

## Architecture

```text
Internet
   ↓
Caddy
   ↓
localhost:3000
   ↓
Docker app container
```

The application container itself is no longer publicly exposed directly.

Instead, Caddy:
- receives public traffic
- manages HTTPS certificates
- forwards requests internally

## What Clicked

The biggest conceptual shift was understanding the role of a reverse proxy.

The browser does not communicate directly with the application container.

Instead:

```text
Browser
   ↓
Reverse proxy
   ↓
Application
```

Caddy also automatically:
- requests certificates
- renews certificates
- redirects HTTP to HTTPS

which makes it extremely beginner-friendly for learning infrastructure.
