# Deployment Guide: Docker + GitHub + Render

This project is a single FastAPI service (see [app.py](app.py)) that serves
both the prediction API and the server-rendered HTML/CSS frontend
(`templates/`, `static/`) — so there is **one Dockerfile**, not separate
backend/frontend containers.

This guide walks through, step by step:

1. Building and running the app in Docker locally
2. Pushing the source code to GitHub
3. Deploying the Docker image to Render

---

## Prerequisites

| Tool | Check it's installed | Install |
|---|---|---|
| Docker Desktop | `docker --version` | https://www.docker.com/products/docker-desktop |
| Git | `git --version` | https://git-scm.com/downloads |
| GitHub account | — | https://github.com/join |

---

## Part 1 — Build and run with Docker (local)

### Step 1.1 — Build the image

From the project root (same folder as `Dockerfile`):

```bash
docker build -t insurance-predictor .
```

What this does:
- `-t insurance-predictor` tags (names) the image so you can refer to it later.
- `.` tells Docker the "build context" (the files it's allowed to `COPY`) is
  the current directory.
- Docker reads `Dockerfile` top to bottom: pulls `python:3.8-slim`, installs
  build tools, installs everything in `requirements.txt`, then copies in
  your code (`app.py`, `templates/`, `static/`, the `.pkl` model file, etc.).

This step can take a few minutes the first time (downloading the base
image + installing `pycaret` and its dependencies). Subsequent builds are
much faster because Docker caches unchanged layers.

### Step 1.2 — Run the container

```bash
docker run -d -p 8000:8000 --name insurance-app insurance-predictor
```

What this does:
- `-d` runs it in the background ("detached").
- `-p 8000:8000` maps port 8000 on your machine to port 8000 inside the
  container (the port `uvicorn` listens on, per the Dockerfile's `CMD`).
- `--name insurance-app` gives the running container a friendly name so
  you can refer to it in later commands instead of a random ID.

### Step 1.3 — Test it

- Open http://localhost:8000 in a browser → you should see the prediction form.
- Open http://localhost:8000/docs → FastAPI's auto-generated interactive API docs.
- Open http://localhost:8000/health → should return `{"status":"ok","model_loaded":true}`.

### Step 1.4 — Useful commands while developing

```bash
docker logs -f insurance-app     # stream the container's logs (Ctrl+C to stop watching)
docker stop insurance-app        # stop the container
docker start insurance-app       # start it again
docker rm -f insurance-app       # stop AND delete the container
docker build -t insurance-predictor .  # rebuild after code changes, then re-run
```

> A container is a *running instance* of an image. If you change `app.py`,
> you must rebuild the image (Step 1.1) and start a new container — a
> running container does not pick up file changes on its own.

---

## Part 2 — Push the code to GitHub

This repo isn't a Git repository yet, so we initialize one first.

### Step 2.1 — Initialize Git

```bash
git init
git add .
git commit -m "Initial commit: FastAPI insurance predictor + Docker setup"
```

The `.gitignore` in this repo already excludes local junk (`__pycache__/`,
virtual envs, editor folders) so it won't get committed.

> **Note on the model file:** `deployment_28042020.pkl` (~22 MB) is under
> GitHub's 100 MB per-file hard limit, so a normal `git add` works fine. If
> you later swap in a much larger model (100 MB+), you'd need
> [Git LFS](https://git-lfs.com/) instead of committing it directly.

### Step 2.2 — If your GitHub repo already exists (with a README)

If you already created the repo on GitHub itself (rather than from this
terminal), and it already contains a `README.md` (GitHub adds one
automatically if you tick "Add a README file"), the remote has a commit
your local repo doesn't know about. Since your local project *also*
already has its own `README.md`, a plain `git push` will be rejected —
Git won't overwrite remote history it hasn't seen.

Copy the remote URL from your GitHub repo page (the green **Code** button),
then run:

```bash
git remote add origin https://github.com/<your-username>/<your-repo>.git
git branch -M main
git pull origin main --allow-unrelated-histories
```

- `remote add origin <url>` tells your local repo where "origin" (GitHub) lives.
- `--allow-unrelated-histories` is needed because your local commit and
  GitHub's initial commit don't share a common ancestor — normally Git
  refuses to merge such unrelated histories, but this is the expected
  situation the first time you connect a pre-existing local project to a
  freshly created GitHub repo.

**If Git reports a conflict in `README.md`** (it likely will, since both
sides have a different version of that file):

1. Open `README.md` in your editor — Git has inserted conflict markers
   that look like:
   ```
   <<<<<<< HEAD
   (your local README content)
   =======
   (GitHub's README content)
   >>>>>>> ...
   ```
2. Edit the file to keep whichever content you want (delete the marker
   lines `<<<<<<<`, `=======`, `>>>>>>>` themselves).
3. Save, then run:
   ```bash
   git add README.md
   git commit -m "Merge remote README with local project"
   ```

If there's no conflict (Git may auto-merge cleanly), skip straight to the
next step.

### Step 2.3 — Push

```bash
git push -u origin main
```

- `-u origin main` links your local `main` branch to GitHub's, so future
  pushes only need `git push`.

From now on, after making changes:

```bash
git add .
git commit -m "describe what changed"
git push
```

---

## Part 3 — Deploy the Docker image on Render

Render can build and run a container directly from your Dockerfile — you
don't need to manually push the image anywhere yourself; Render does the
`docker build` for you from the GitHub repo.

### Step 3.1 — Create the Render service

1. Go to https://dashboard.render.com/ and log in (or sign up).
2. Click **New +** → **Web Service**.
3. Connect your GitHub account if prompted, then select the
   `insurance-prediction` repo you just pushed.

### Step 3.2 — Configure it

- **Environment**: choose **Docker** (Render detects the `Dockerfile` in
  the repo root automatically).
- **Instance type**: the free tier is enough to try this out.
- Render automatically sets a `$PORT` environment variable and expects
  your app to listen on it. Our `Dockerfile` hardcodes `--port 8000`, so
  either:
  - **Option A (simplest):** in the Render dashboard, set the service's
    port to `8000` under **Settings → Port** so Render forwards traffic
    to the port the container actually listens on, or
  - **Option B (matches Render's convention):** change the Dockerfile's
    `CMD` to read the port from the environment at container start:
    ```dockerfile
    CMD ["sh", "-c", "uvicorn app:app --host 0.0.0.0 --port ${PORT:-8000}"]
    ```
    This uses Render's `$PORT` if it's set, and falls back to 8000 for
    local `docker run`.

### Step 3.3 — Deploy

Click **Create Web Service**. Render will:
1. Pull your GitHub repo.
2. Run `docker build` using your `Dockerfile`.
3. Start the container and expose it at a public `https://<your-service>.onrender.com` URL.

### Step 3.4 — Redeploying after changes

Every `git push` to the connected branch triggers Render to automatically
rebuild and redeploy — this is the same "one code path from laptop to
production" idea that Docker is designed to give you.

### Step 3.5 — Checking it's healthy

Visit `https://<your-service>.onrender.com/health`. Render also lets you
configure this exact path as a **Health Check Path** under service
Settings, so Render itself will restart the service automatically if it
ever stops responding correctly.

---

## Recap of the full pipeline

```
local code  →  docker build/run (test locally)  →  git push (GitHub)  →  Render builds & deploys the same Dockerfile
```

The key MLOps idea here: the **same Dockerfile** is used for local testing
and production deployment, so what you verified on your machine is exactly
what runs in production — no environment drift.
