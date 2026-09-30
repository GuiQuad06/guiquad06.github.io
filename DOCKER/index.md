---
layout: default
title: Docker
---

# Docker

- [Overview](#overview)
- [First project](#first-project)
- [Cheatsheets](#cheatsheets)
- [Yocto in Docker with CROPS](#yocto-in-docker-with-crops)

---

## Overview

- **Docker image** — a snapshot of a filesystem.
- **Docker container** — a running instance of the image.
- **Tag** — a label applied to an image to identify version, purpose, environment
  (`docker build -t` or `docker tag`).
- **Labels** — a sticky note attaching metadata (key/value), appended in the Dockerfile.
- To associate a remote with an image, use `docker tag`.
- In a Dockerfile, avoid multiplying layers: use `\` and `&&` to continue a `RUN`
  instruction on the next line instead of stacking several `RUN` instructions.

---

## First project

**Prerequisites:** Docker (Desktop) + a Dockerfile at the root of the project. The
Dockerfile specifies dependencies, environment variables, the run command, etc.
(`FROM` sets the base image.)

### 1. Building the image

```bash
docker build --force-rm=true .
docker build --force-rm=true . --no-cache   # force a full rebuild
docker image pull python
docker build --force-rm=true . -t toto:v1   # tag the image
```

Use `docker push` / `docker pull` to interact with a registry (tag the image first to
map it to a registry + version).

### 2. Running the image

```bash
docker run           # launch the container
docker start / stop  # start / stop the app
docker image prune   # clean up dangling images
```

- Create volumes to persist data.
- Use `docker compose` for multi-container apps.

---

## Cheatsheets

- [Docker: Your First Project](https://github.com/LinkedInLearning/docker-your-first-project-4485003)
- [Docker: Build and Optimize Docker Images](https://github.com/LinkedInLearning/docker-build-and-optimize-docker-images-3978299)
- [Docker for Developers: Create and Manage Docker Containers](https://github.com/LinkedInLearning/docker-for-developers-create-and-manage-docker-containers-4009272)

---

## Yocto in Docker with CROPS

`crops/poky` — Docker Hub repository providing ready-to-use Yocto/Poky build containers.
See [Yocto x Docker : `crops/poky` image (Debian tags)](../EMBEDDED_LINUX/crops_poky_debian.html)
for a full walkthrough.

[Back to home]({{ site.baseurl }}/)
