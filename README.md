# NGINX Learning Notes 🌐

Notes from learning NGINX — what it is, its core features, and how its configuration model works. Sharing here as a quick reference for anyone else picking up NGINX.

---

## 📑 Table of Contents
- [What is NGINX?](#-what-is-nginx)
- [Core Features](#-core-features)
- [Process Model: Master vs. Worker](#-process-model-master-vs-worker)
- [Configuration Lifecycle: sites-available vs. sites-enabled](#-configuration-lifecycle-sites-available-vs-sites-enabled)
- [Basic Config Example](#-basic-config-example)
- [Useful NGINX Commands](#-useful-nginx-commands)
- [Topics Worth Adding to Your Notes](#-topics-worth-adding-to-your-notes)

---

## 🖥 What is NGINX?

**NGINX** is a simple, high-performance **web server** used to serve web files — but in practice it's used for much more than that, thanks to its features around proxying, load balancing, and caching.

---

## ⚙️ Core Features

- **Reverse Proxy** — sits in front of one or more backend servers and forwards client requests to them, hiding the backend's details from the client.
- **Load Balancing** — distributes incoming traffic across multiple backend servers (round-robin, least connections, IP hash, etc.) to improve reliability and performance.
- **URL Director (Rewrites/Redirects)** — routes or rewrites requests based on URL patterns.
- **Indexing** — serves a default file (e.g., `index.html`) when a directory is requested.
- **Caching** — stores responses so repeated requests can be served faster without hitting the backend every time.

---

## 🧵 Process Model: Master vs. Worker

NGINX runs using two types of processes:

- **Master Process** — reads and validates the configuration, and manages the worker processes (starting, stopping, reloading them).
- **Worker (Child) Processes** — do the actual work: handling reverse proxying, load balancing, indexing, and caching for incoming requests.

This split is why NGINX can **reload its config with zero downtime** — the master process spins up new workers with the updated config while gracefully finishing off requests on the old ones.

---

## 🔁 Configuration Lifecycle: sites-available vs. sites-enabled

- **`sites-available/`** — holds all your site configuration files, whether they're currently live or not. Think of it as your library of configs.
- **`sites-enabled/`** — holds only the configs that are actually **active/deployed**. Files here are typically **symlinks** pointing back to files in `sites-available/`.

**Workflow:**
1. Write/edit your site's config in `sites-available/`.
2. Create a symlink for it inside `sites-enabled/` to deploy it.
3. To edit a live site, you edit the file in `sites-available/` (since the symlink just points there) and reload NGINX for the changes to take effect.

This separation lets you keep configs ready to go without them automatically going live, and makes it easy to enable/disable a site by just adding/removing a symlink.

```bash
# Enable a site
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/

# Disable a site
sudo rm /etc/nginx/sites-enabled/mysite

# Reload NGINX to apply changes
sudo nginx -t && sudo systemctl reload nginx
```

---

## 📝 Basic Config Example

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        root /var/www/html;
        index index.html;
    }

    location /api/ {
        proxy_pass http://localhost:3000/;
    }
}
```

- `server { }` — defines a **server block** (a virtual host).
- `listen` — the port NGINX listens on.
- `server_name` — the domain(s) this block responds to.
- `location` — matches request paths and defines how to handle them (serve static files, proxy to a backend, etc.).
- `proxy_pass` — forwards matching requests to a backend server (this is the reverse-proxy piece in action).

---

## 💻 Useful NGINX Commands

| Command | Purpose |
|---|---|
| `nginx -t` | Test configuration for syntax errors before applying |
| `nginx -s reload` / `systemctl reload nginx` | Reload config without downtime |
| `systemctl start/stop/restart nginx` | Manage the NGINX service |
| `nginx -v` / `nginx -V` | Check version / compiled modules |
| `tail -f /var/log/nginx/access.log` | Watch access logs live |
| `tail -f /var/log/nginx/error.log` | Watch error logs live |

---

## 🧠 Topics Worth Adding to Your Notes

A few closely related areas that build naturally on what's above:

- **`nginx.conf` structure** — the main config file that sets global directives and includes everything in `sites-enabled/` (or `conf.d/`) via an `include` directive.
- **SSL/TLS termination** — NGINX is commonly used to handle HTTPS (via `listen 443 ssl;` + certificate directives), decrypting traffic before passing plain HTTP to backend services — a very common reverse-proxy use case.
- **Gzip/Brotli compression** — enabling `gzip on;` reduces response size and speeds up page loads.
- **Rate limiting** — `limit_req_zone` / `limit_req` directives protect backends from being overwhelmed by too many requests from a single client.
- **Upstream blocks** — the `upstream { }` block is where you actually define the pool of backend servers used for load balancing, referenced by `proxy_pass`.
- **Static vs. dynamic content handling** — NGINX excels at serving static files directly (fast, no backend hit) while proxying dynamic requests (e.g., API calls) to app servers like Django, Node, or FastAPI.
- **Logging & log formats** — customizing `access_log`/`error_log` formats for better observability and integration with monitoring tools.
- **NGINX as part of a container stack** — a very common pattern (as in your Docker notes) is NGINX as a reverse proxy in front of an app server and database, all wired together via Docker Compose.
- **Security headers** — directives like `add_header X-Frame-Options`, `Strict-Transport-Security`, etc., to harden responses against common web attacks.
- **NGINX vs. NGINX Plus** — the open-source core vs. the commercial version with extra features (active health checks, session persistence, a management dashboard).

---

## 🛠 About These Notes

Compiled while learning NGINX as part of broader DevOps/cloud learning, alongside Docker and container networking. Feel free to fork, use, or suggest corrections via a PR/issue!

**Topics covered:** `Reverse Proxy` · `Load Balancing` · `Master/Worker Processes` · `sites-available` / `sites-enabled` · `Basic Config`
