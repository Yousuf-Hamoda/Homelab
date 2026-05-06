# 🏠 Homelab Docker Stacks

A collection of self-hosted services managed via [Portainer](https://www.portainer.io/) on a Linux server. Each stack is a standalone Docker Compose file ready to deploy.

---

## 📦 Stacks

| Stack | Description | Port |
|---|---|---|
| [Actual Budget](#actual-budget) | Local-first personal finance & budgeting | 5006 |
| [Audiobookshelf](#audiobookshelf) | Audiobook & podcast server with mobile apps | 13378 |
| [Immich](#immich) | Self-hosted Google Photos alternative | 2283 |
| [Kavita](#kavita) | Manga, comics, and ebook reader | 5000 |
| [LocalAI](#localai) | Local LLM inference server (OpenAI-compatible API) | 8181 |
| [Vikunja](#vikunja) | Open-source to-do & project management app | 3456 |
| [VS Code Server](#vs-code-server) | Browser-based VS Code via code-server | 8443 |

---

## 🚀 Fresh Install Guide

### 1. Install Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

### 2. Install Portainer

```bash
docker volume create portainer_data

docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

Then open `https://YOUR_SERVER_IP:9443` in your browser and create your admin account.

### 3. Create your folder structure

Replace `YOUR_USERNAME` with your Linux username throughout:

```bash
# Container config storage
mkdir -p ~/homelab/containers

# Data / media storage
mkdir -p ~/homelab/data
```

### 4. Create a Cloudflare network (required by some stacks)

Several stacks use an external Docker network called `cloudflare` for use with Cloudflare Tunnels. Create it once on the host:

```bash
docker network create cloudflare
```

> If you don't use Cloudflare Tunnels, remove the `networks` sections from the relevant compose files.

### 5. Deploy a stack in Portainer

1. Open Portainer → **Stacks** → **Add stack**
2. Give it a name (e.g. `kavita`)
3. Paste the contents of the relevant `.yaml` file into the editor
4. Replace all `YOUR_USERNAME` placeholders with your actual Linux username
5. Fill in any other required values (see per-stack notes below)
6. Click **Deploy the stack**

---

## Stack Details

### Actual Budget

**File:** `actual-budget.yaml`

A fast, privacy-focused budgeting app that runs entirely locally. No accounts or cloud sync required.

- **URL:** `http://YOUR_SERVER_IP:5006`
- **Data:** `~/homelab/data/actual-budget`
- **No extra configuration required** — just deploy and open in your browser.

---

### Audiobookshelf

**File:** `audiobookshelf.yaml`

A self-hosted audiobook and podcast server with iOS/Android apps for streaming on the go.

- **URL:** `http://YOUR_SERVER_IP:13378`
- **Requires:** Cloudflare network (`docker network create cloudflare`)
- **Mount your media** at `~/homelab/data/audiobooks` and `~/homelab/data/podcasts`
- **Change `TZ`** to your timezone (e.g. `America/New_York`). Full list: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones

---

### Immich

**File:** `immich.yaml`

A full-featured self-hosted photo and video backup solution — a true Google Photos replacement with face recognition, albums, and mobile apps.

- **URL:** `http://YOUR_SERVER_IP:2283`
- **Requires:** A `stack.env` file — in Portainer, add these under **Environment variables** or use the **stack.env** option:

```env
UPLOAD_LOCATION=/home/YOUR_USERNAME/homelab/data/immich
DB_DATA_LOCATION=/home/YOUR_USERNAME/homelab/data/immich-db
DB_PASSWORD=CHANGE_ME_STRONG_PASSWORD
DB_USERNAME=immich
DB_DATABASE_NAME=immich
IMMICH_VERSION=release
```

> Generate a strong DB password with: `openssl rand -hex 24`

---

### Kavita

**File:** `kavita.yaml`

A self-hosted manga, comics, and ebook reader with a clean web UI and OPDS support for e-readers.

- **URL:** `http://YOUR_SERVER_IP:5000`
- **Requires:** Cloudflare network (`docker network create cloudflare`)
- **Mount your media** at `~/homelab/data/manga`, `~/homelab/data/comics`, `~/homelab/data/books`
- **Change `TZ`** to your timezone

---

### LocalAI

**File:** `localai.yaml`

A self-hosted, OpenAI-compatible API server for running LLMs and other AI models locally — no GPU required for CPU mode.

- **URL:** `http://YOUR_SERVER_IP:8181`
- **Models directory:** `~/homelab/data/localai/models`

#### Switching to GPU mode

1. In Portainer's editor, comment out the CPU image line and uncomment the GPU image line
2. Uncomment the entire `deploy:` block at the bottom
3. Redeploy the stack

> GPU mode requires the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) installed on the host.

---

### Vikunja

**File:** `vikunja.yaml`

A self-hosted to-do list and project management app with kanban boards, reminders, and team sharing.

- **URL:** `http://YOUR_SERVER_IP:3456`
- **Requires:** Cloudflare network (`docker network create cloudflare`)
- **Before deploying, set these values in the compose file:**

| Variable | What to set |
|---|---|
| `VIKUNJA_SERVICE_PUBLICURL` | Your server IP or domain, e.g. `http://192.168.1.100:3456` |
| `VIKUNJA_SERVICE_JWTSECRET` | A long random string — generate with: `openssl rand -hex 32` |

---

### VS Code Server

**File:** `vscode.yaml`

Run VS Code in your browser via [code-server](https://github.com/coder/code-server). Access your full development environment from any device.

- **URL:** `http://YOUR_SERVER_IP:8443`
- **Your home directory and Code folder** are mounted directly into the container
- **Check your UID:GID** with `id YOUR_USERNAME` and update the `user:` field if needed (default is `1000:1000`)

---

## 🔒 Security Notes

- **Never commit real secrets** to this repo. All sensitive values use `CHANGE_ME_*` placeholders.
- For `VIKUNJA_SERVICE_JWTSECRET` and `DB_PASSWORD`, generate strong values with `openssl rand -hex 32` before deploying.
- Consider putting services behind a reverse proxy (e.g. Cloudflare Tunnel or Nginx Proxy Manager) rather than exposing ports directly to the internet.
- The `cloudflare` external network is intended for use with Cloudflare Tunnels. Remove it from stacks if you don't use this.

---
