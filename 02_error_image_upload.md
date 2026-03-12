That’s the cause 👍

Your image upload is failing because Django is trying to write uploaded files into:

```text
/app/media
```

but the container user does **not have write permission** there.

## Why this happens

In your dev setup, you likely have:

```yaml
volumes:
  - .:/app
```

That bind mount replaces the `/app` directory from the image with your local project folder.
So even if the Dockerfile set permissions correctly during build, the running container sees the host-mounted folder instead.

As a result, `/app/media` may exist but not be writable by the `wagtail` user inside the container.

---

# Best fix for development

Use a **separate writable volume for media** in `docker-compose.dev.yml`.

## Change your `web` service

Instead of only:

```yaml
volumes:
  - .:/app
```

use:

```yaml
volumes:
  - .:/app
  - dmlibrary_media_dev:/app/media
```

If you also want collected static separated cleanly, you can add:

```yaml
  - dmlibrary_static_dev:/app/staticfiles
```

So the full `web` section becomes something like:

```yaml
web:
  build:
    context: .
  container_name: dmlibrary-dev-web
  restart: unless-stopped
  command: python manage.py runserver 0.0.0.0:8000
  env_file:
    - .env.dev
  environment:
    DATABASE_URL: postgres://dmlibrary_user:dmlibrary_password@db:5432/dmlibrary_dev
    DJANGO_SETTINGS_MODULE: mysite.settings.dev
  volumes:
    - .:/app
    - dmlibrary_media_dev:/app/media
    - dmlibrary_static_dev:/app/staticfiles
  depends_on:
    - db
  ports:
    - "8000:8000"
```

And at the bottom:

```yaml
volumes:
  postgres_dev_data:
  pgadmin_data:
  dmlibrary_media_dev:
  dmlibrary_static_dev:
```

---

# Then restart

Run:

```bash
docker compose -f docker-compose.dev.yml down
docker compose -f docker-compose.dev.yml up -d --build
```

Then try uploading again.

---

# Simpler temporary fix

If you want a quick test without editing too much, inside WSL in your project folder:

```bash
mkdir -p media
chmod 777 media
```

Then restart the web container:

```bash
docker compose -f docker-compose.dev.yml restart web
```

This often works for local dev, but I recommend the **named volume solution** above because it is cleaner and more reliable.

---

# Why named volume is better

It avoids permission problems from WSL/Windows bind mounts and keeps uploaded files persistent.

---

# Recommendation

Do this now:

1. add `dmlibrary_media_dev:/app/media` to `web.volumes`
2. add `dmlibrary_media_dev:` under top-level `volumes`
3. restart compose
4. test image upload again

If you want, paste your current `docker-compose.dev.yml` and I’ll rewrite just the exact lines to change.

---

That means `/app/media` now exists, but the container still cannot **create the subfolder**:

```text
/app/media/original_images
```

So the problem is still **ownership/permissions** on the mounted media directory.

## Best fix for dev

The simplest reliable fix is to make the Django container run as **root in development only**.

In `docker-compose.dev.yml`, add this to the `web` service:

```yaml
user: "0:0"
```

So it becomes like:

```yaml
web:
  build:
    context: .
  container_name: dmlibrary-dev-web
  restart: unless-stopped
  user: "0:0"
  command: python manage.py runserver 0.0.0.0:8000
  env_file:
    - .env.dev
  environment:
    DATABASE_URL: postgres://dmlibrary_user:dmlibrary_password@db:5432/dmlibrary_dev
    DJANGO_SETTINGS_MODULE: mysite.settings.dev
  volumes:
    - .:/app
    - dmlibrary_media_dev:/app/media
    - dmlibrary_static_dev:/app/staticfiles
  depends_on:
    - db
  ports:
    - "8000:8000"
```

Then restart:

```bash
docker compose -f docker-compose.dev.yml down
docker compose -f docker-compose.dev.yml up -d --build
```

This is acceptable for **local development**. I would not do this in production.

---

## Why this happened

Your Dockerfile runs as the `wagtail` user, which is good for production.
But in dev, mounted volumes often end up owned by `root`, and the `wagtail` user cannot write inside them.

That is why Wagtail can see `/app/media` but cannot create:

```text
/app/media/original_images
```

---

## Alternative fix

If you want to keep the non-root user, you would need to `chown` the media directory inside the container before startup, for example with an entrypoint script.

But for dev, `user: "0:0"` is the fastest clean fix.

---

## After changing it

Try uploading again.
If it still fails, run:

```bash
docker compose -f docker-compose.dev.yml exec web ls -ld /app/media
docker compose -f docker-compose.dev.yml exec web whoami
```

But I expect `user: "0:0"` to solve this.

---

```bash
docker compose -f docker-compose.dev.yml exec web ls -ld /app/media
docker compose -f docker-compose.dev.yml exec web whoami

drwxr-xr-x 2 root root 4096 Mar 12 10:10 /app/media
root
```
