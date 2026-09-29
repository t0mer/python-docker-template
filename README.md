# python-docker-template

A minimal GitHub template repository for starting a Dockerized Python application. It gives you
a Dockerfile that installs Python on Ubuntu and runs `app/app.py`, a skeleton
`docker-compose.yaml`, a `requirements.txt` and a `VERSION` file. You bring the code.

> **Note:** `app/app.py` is an empty placeholder. The image builds an environment for your
> script, but the template itself contains no application logic.

> **Warning:** the unmodified template does not build today (end-of-life base image and
> PEP 668). See [Notes and known limitations](#notes-and-known-limitations) for the fixes.

## What's included

| File | Purpose |
|------|---------|
| `Dockerfile` | Builds an Ubuntu-based image with Python 3 and pip, installs `requirements.txt`, copies `app/` into `/app` and runs `app/app.py`. |
| `docker-compose.yaml` | Skeleton Compose file with placeholders for the service name, image and container name, plus an example environment variable and a bind mount of `./app`. It must be filled in before use. |
| `requirements.txt` | Python dependencies installed into the image. Contains only `loguru` (unpinned). |
| `app/app.py` | Entry point run by the container. Empty. |
| `VERSION` | Version string (`0.1.1`). Nothing in the repository reads it. |
| `LICENSE` | Apache License 2.0. |

### Dockerfile in detail

- **Base image:** `ubuntu:24.10`
- **Labels:** `maintainer=""` (empty, fill it in)
- **Environment:** `PYTHONIOENCODING=utf-8`, `LANG=C.UTF-8`
- **System packages (apt):** `python3-pip`, `libffi-dev`, `libssl-dev`
- **pip:** upgrades `pip` and `setuptools`, then installs `requirements.txt` (copied to `/tmp`)
- **Copies:** the `app/` directory to `/app`
- **Working directory:** `/app`
- **Entrypoint:** `/usr/bin/python3 /app/app.py`
- No `EXPOSE`, `USER`, `HEALTHCHECK` or `CMD` instructions.

### docker-compose.yaml in detail

- `version: "3.6"`
- One service whose key is a placeholder (`container_name: #Change to your name`) that you
  replace with your service name.
- `image:` and `container_name:` are empty.
- `restart: always`
- `environment`: `EXAMPLE_ENV=` (an example variable; replace or remove it)
- `volumes`: `./app:/app`, which mounts your local code over the code baked into the image,
  so edits apply on container restart without rebuilding.
- No `build:` section and no `ports:`.

## Using the template

1. Create a new repository from the template:
   - On GitHub, click **Use this template** → **Create a new repository**, or
   - With the GitHub CLI:
     ```bash
     gh repo create my-app --template t0mer/python-docker-template --private --clone
     ```
2. Write your application in `app/app.py`. Additional modules can go next to it in `app/`;
   the whole directory is copied into the image.
3. Add dependencies to `requirements.txt`, one per line (pin versions, e.g. `loguru==0.7.2`).
   Remove `loguru` if you don't use it.
4. Fill in `LABEL maintainer` in the `Dockerfile` and bump `VERSION` if you use it.
5. Fill in `docker-compose.yaml` (see below).

## Building and running

> **Warning:** as shipped, `docker build` fails. Fix the base image and the pip install first;
> see [Notes and known limitations](#notes-and-known-limitations).

With Docker:

```bash
docker build -t my-app .
docker run --rm my-app
```

With Docker Compose, first replace the placeholders. A filled-in example that builds the
image locally:

```yaml
services:
  my-app:
    build: .
    image: my-app:latest
    container_name: my-app
    restart: always
    environment:
      - EXAMPLE_ENV=value
    volumes:
      - ./app:/app
```

```bash
docker compose up -d --build
docker compose logs -f
```

If your app listens on a port, add a `ports:` mapping to the Compose file (and optionally
`EXPOSE` in the Dockerfile for documentation).

## Customizing

- **Environment variables:** add them under `environment:` in the Compose file (or `-e` with
  `docker run`) and read them in your code with `os.environ`.
- **System packages:** add them to the `apt install` step in the Dockerfile.
- **Different entry point:** change the `ENTRYPOINT` line.
- **Versioning:** `VERSION` is not wired into the build. If you want it, pass it as a build arg
  or read it in your code.

## Notes and known limitations

These are properties of the template as it stands; review them before using it for anything
beyond a quick prototype.

- **The build fails: end-of-life base image.** `ubuntu:24.10` (oracular) is a non-LTS release
  that reached end of life in July 2025. Its repositories have moved to
  `old-releases.ubuntu.com`, and `archive.ubuntu.com` returns 404 for them, so `apt update`
  fails. Switch to a supported base image, such as `python:3.12-slim` or an LTS tag like
  `ubuntu:24.04`.
- **The build fails: pip on recent Ubuntu.** Even with apt fixed, Ubuntu 23.04 and later mark the
  system Python as "externally managed" (PEP 668), so the `pip3 install` steps fail with
  `externally-managed-environment`. Use a base image with its own Python (e.g.
  `python:3.12-slim`, where pip works directly) or install into a virtual environment.
- **Runs as root.** There is no `USER` instruction, so the app runs as root inside the container.
  Add a non-root user for anything exposed to a network.
- **No healthcheck.** Add a `HEALTHCHECK` if your app is a long-running service.
- **Unpinned versions.** `requirements.txt` is unpinned and pip/setuptools are always upgraded to
  the latest release, so builds are not reproducible.
- **Image size and caching.** `apt update` runs in its own layer and apt lists are not cleaned up;
  `requirements.txt` is installed without `--no-cache-dir`. There is no `.dockerignore`.
- **Compose file is not valid as shipped.** The service name, `image` and `container_name` are
  placeholders, and the top-level `version` key is obsolete in Docker Compose v2. The
  `EXAMPLE_ENV` comment ("Google account email") is left over from another project.
- **Bind mount overrides the image.** `./app:/app` means the container runs your local files,
  not the ones built into the image. Remove the volume for production.
- **No CI.** There are no GitHub Actions workflows for building or publishing the image.

## License

Licensed under the [Apache License 2.0](LICENSE).
