---
title: "Understanding Docker Volumes and PostgreSQL"
date: 2026-05-10
draft: false
tags:
  - docker
  - postgres
  - devops
categories:
  - Learning Notes
description: "What finally made Docker volumes and PostgreSQL persistence click for me."
---

# Understanding Docker Volumes and PostgreSQL

One of the most confusing parts of my first deployment was PostgreSQL authentication failing even after changing the password in `.env`.

The important lesson:

```text
PostgreSQL only uses POSTGRES_PASSWORD the first time the database is created.
```

## What Happened

I originally started the database container with password A.

Docker created a persistent volume:

```text
dsu_pg_data
```

Later I changed the password in `.env` to password B.

The application then failed with:

```text
password authentication failed for user "postgres"
```

## Why?

Because the database volume still contained the original database initialized with password A.

Changing `.env` does not reconfigure an existing PostgreSQL database automatically.

## Fix

The learning-stage fix was:

```bash
sudo docker compose down -v
sudo docker compose up -d
```

The `-v` removed the existing PostgreSQL volume and allowed the database to initialize again with the new password.

## Important Concept

Containers are disposable.

Volumes are persistent.

That distinction finally made Docker persistence much easier to understand.
