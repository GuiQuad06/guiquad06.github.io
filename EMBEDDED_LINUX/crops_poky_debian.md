---
layout: default
title: crops/poky (Debian tags)
---

# Using `crops/poky` (Debian tags)

Notes based on:

- <https://hub.docker.com/r/crops/poky>
- <https://github.com/crops/poky-container>

## 1. What it is

`crops/poky` is a prebuilt image containing all host dependencies needed to run
**BitBake / Poky / OpenEmbedded** (the Yocto Project build system).

Its distinguishing feature versus a plain "Yocto dockerfile" is a set of helper
scripts (`poky-entry.py`, `usersetup.py`, `poky-launch.sh`) that **create a user
inside the container with the same uid/gid as the mounted work directory**. The
build output on the host therefore belongs to you, not to `root`.

The image is built from `crops/yocto:$BASE_DISTRO-base`, so one image exists per
host distro flavour. You pick the flavour with the **tag**.

## 2. Available Debian tags

| Tag | Base |
| --- | --- |
| `crops/poky:debian-12` | Debian 12 (bookworm) — most recent |
| `crops/poky:debian-11` | Debian 11 (bullseye) |
| `crops/poky:debian-10` | Debian 10 (buster) |
| `crops/poky:debian-9`  | Debian 9 (stretch) — stale |

Other families exist too: `ubuntu-18.04/20.04/22.04`, `fedora-36..40`,
`opensuse-15.2..15.6`, `alma-8/9`, `centos-7/8`, and `latest`.

> There is no bare `debian` tag — you must use the versioned form, e.g.
> `debian-12`.

Listing tags yourself:

```powershell
(Invoke-RestMethod "https://hub.docker.com/v2/repositories/crops/poky/tags?page_size=100").results |
  Select-Object name, last_updated
```

## 3. Pull or run directly?

You do **not** have to pull explicitly — `docker run` pulls the image on first
use. Pulling separately is only useful to pre-fetch (~600 MB) or to refresh:

```bash
docker pull crops/poky:debian-12
```

You also do **not** build anything yourself. The image is ready to use; the
`Dockerfile` in the GitHub repo is only for maintainers (it requires
`--build-arg BASE_DISTRO=debian-12`).

## 4. Running it

### 4.1 Create a work directory (Linux)

The work directory holds all build output. **You must own it** — its uid/gid
becomes the uid/gid of the user inside the container.

```bash
mkdir -p /home/myuser/yocto
```

### 4.2 Linux

```bash
docker run --rm -it \
  -v /home/myuser/yocto:/workdir \
  crops/poky:debian-12 --workdir=/workdir
```

With SELinux in enforcing mode, add the `:Z` label:

```bash
docker run --rm -it \
  -v /home/myuser/yocto:/workdir:Z \
  crops/poky:debian-12 --workdir=/workdir
```

### 4.3 Windows / macOS

Use a **named Docker volume** instead of a bind mount (bind mounts from
NTFS/APFS break Yocto's permission and hardlink requirements):

```powershell
docker volume create myvolume
docker run --rm -it -v myvolume:/workdir crops/poky:debian-12 --workdir=/workdir
```

See <https://github.com/crops/docker-win-mac-docs/wiki> for the full Windows/Mac
setup.

You should land on a prompt like:

```text
pokyuser@3bbac563cacd:/workdir$
```

### 4.4 What the flags mean

| Flag | Meaning |
| --- | --- |
| `--rm` | delete the container when you exit (the volume/bind mount keeps the data) |
| `-it` | interactive TTY — you need a shell |
| `-v host:/workdir` | mount the persistent build directory |
| `--workdir=/workdir` | **argument to the image entrypoint**, not to Docker: sets the starting directory *and* derives the container user's uid/gid from that directory |

Everything placed after the image name is passed to `poky-entry.py`.

## 5. Entrypoint arguments (`poky-entry.py`)

| Argument | Description |
| --- | --- |
| `--workdir=PATH` | Active directory once the container runs. Default `/home/pokyuser`. Without `--id`, its uid/gid are reused for the container user. |
| `--id=uid:gid` | Force the uid/gid of the in-container user instead of deriving it from the workdir. |
| `cmd ...` | Trailing command to execute after setup instead of an interactive shell (mostly used for tests/CI). |

Examples:

```bash
# Force uid/gid explicitly
docker run --rm -it -v $PWD/yocto:/workdir \
  crops/poky:debian-12 --workdir=/workdir --id=1000:1000

# Non-interactive: run one command and exit
docker run --rm -v $PWD/yocto:/workdir \
  crops/poky:debian-12 --workdir=/workdir bash -c "cd poky && ls"
```

Note: if you use Docker's own `-w /workdir` flag, the entrypoint detects the
current directory and behaves as if `--workdir` had been given.

## 6. Typical first build inside the container

```bash
cd /workdir
git clone -b scarthgap git://git.yoctoproject.org/poky
cd poky
source oe-init-build-env            # creates and enters build/
bitbake core-image-minimal
runqemu qemux86-64                  # needs extra host setup / KVM
```

Follow <https://docs.yoctoproject.org/brief-yoctoprojectqs/index.html> from that
prompt.

## 7. Practical tips

- **Persist `downloads/` and `sstate-cache/`** outside the build tree so rebuilds
  and new containers reuse them. In `conf/local.conf`:

  ```conf
  DL_DIR = "/workdir/downloads"
  SSTATE_DIR = "/workdir/sstate-cache"
  ```

- **Disk space**: a single image build easily needs 50–100 GB. On Docker Desktop,
  raise the VM disk size, CPU and RAM limits (Yocto is very parallel).
- **Do not run as root** and do not add `--user` to `docker run`; the entrypoint
  handles user creation. Overriding it breaks the uid/gid matching.
- **Do not override `--entrypoint`** unless you know what you are doing — you
  would lose `dumb-init` (zombie reaping) and the user setup.
- **Keep the container alive across sessions** by dropping `--rm` and using
  `--name`:

  ```bash
  docker run -it --name yocto -v /home/myuser/yocto:/workdir crops/poky:debian-12 --workdir=/workdir
  # later
  docker start -ai yocto
  ```

- **Tag pinning**: prefer `debian-12` over `latest` so your build host stays
  reproducible.

## 8. Compose equivalent

```yaml
services:
  poky:
    image: crops/poky:debian-12
    command: ["--workdir=/workdir"]
    stdin_open: true
    tty: true
    volumes:
      - yocto:/workdir

volumes:
  yocto:
```

Run with:

```bash
docker compose run --rm poky
```

[Back to Embedded Linux](./)
