# maarten.cv

Personal CV site. Plain HTML/CSS, no build step, served by nginx in a small
Docker container. GitHub Actions deploys it to a Raspberry Pi over SSH on
every push to `main`; the Pi's existing Caddy instance reverse-proxies to it.

## Local dev

Just open `site/index.html` in a browser, or run it through the same
container you'll deploy:

```
docker compose up --build
```

Then visit http://localhost:8085.

## One-time setup on the Pi

```
git clone <this repo url> ~/apps/maarten-cv
cd ~/apps/maarten-cv
docker compose up -d --build
```

Add a site block to your Caddyfile:

```
maarten.cv {
    reverse_proxy 127.0.0.1:8085
}
```

Reload Caddy (`caddy reload` or `systemctl reload caddy`, depending on your
setup).

## CI/CD

`.github/workflows/deploy.yml` runs on every push to `main`: it SSHes into
the Pi, `git pull`s, and runs `docker compose up -d --build`. It needs these
repo secrets (Settings → Secrets and variables → Actions):

| Secret            | Value                                              |
|--------------------|-----------------------------------------------------|
| `PI_HOST`          | Pi's hostname or IP reachable from the internet      |
| `PI_USER`          | SSH user on the Pi                                   |
| `PI_SSH_KEY`       | Private key for a deploy key with SSH access to the Pi |
| `PI_PORT`          | SSH port (optional, defaults to 22)                  |
| `PI_DEPLOY_PATH`   | Path to the repo clone on the Pi, e.g. `/home/pi/apps/maarten-cv` |

Generate a dedicated deploy key rather than reusing a personal one:

```
ssh-keygen -t ed25519 -f deploy_key -N ""
```

Add `deploy_key.pub` to `~/.ssh/authorized_keys` on the Pi, and the contents
of `deploy_key` (private) as the `PI_SSH_KEY` secret.

## Updating content

Edit `site/index.html`, commit, push to `main`. That's it.
