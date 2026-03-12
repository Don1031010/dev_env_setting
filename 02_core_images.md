Yes, まりさん 👍
For **`core_images`**, you will follow **almost the same process** as `cms_taxonomy`, but there are **two extra steps specific to Wagtail images**.

This app defines a **custom Wagtail image model**, so Django/Wagtail must be told to use it.

---

# Phase 2: Import `core_images`

## Step 1 — confirm files copied

In `dmlibrary_local` you should now have:

```
core_images/
    __init__.py
    models.py
    admin.py
    apps.py
    migrations/
```

Check:

```bash
ls core_images
```

---

# Step 2 — add to `INSTALLED_APPS`

Open:

```
mysite/settings/base.py
```

Add:

```python
"core_images",
```

For example:

```python
INSTALLED_APPS = [
    ...
    "cms_taxonomy",
    "core_images",
]
```

---

# Step 3 — tell Wagtail to use the custom image model

This is the **important step that cms_taxonomy did not require**.

In `base.py` add:

```python
WAGTAILIMAGES_IMAGE_MODEL = "core_images.CustomImage"
```

Put it near the other Wagtail settings.

Example:

```python
WAGTAIL_SITE_NAME = "mysite"

WAGTAILIMAGES_IMAGE_MODEL = "core_images.CustomImage"
```

---

# Step 4 — run migrations

Because this app creates a **replacement Wagtail image table**, migrations are required.

Run:

```bash
docker compose -f docker-compose.dev.yml exec web python manage.py migrate
```

If Django complains about missing migrations:

```bash
docker compose -f docker-compose.dev.yml exec web python manage.py makemigrations core_images
docker compose -f docker-compose.dev.yml exec web python manage.py migrate
```

---

# Step 5 — verify in admin

Open:

```
http://localhost:8000/admin
```

Go to:

```
Images
```

Upload a test image.

If it works, the custom image model is active.

---

# Step 6 — verify in Django shell (optional)

```bash
docker compose -f docker-compose.dev.yml exec web python manage.py shell
```

Then:

```python
from wagtail.images import get_image_model
get_image_model()
```

Expected result:

```
core_images.CustomImage
```

---

# Step 7 — commit the step

```bash
git add core_images
git commit -m "Import core_images custom Wagtail image model"
git push
```

---

# ⚠️ Important warning

The **custom image model must be installed before any app that references images**.

That includes:

```
photo
book
blog
```

So your order is correct:

```
cms_taxonomy
core_images
book
photo
blog
core
```

---

# After this step

Next we should import **`book`**, which is the first real content app.

It will introduce:

* `BookIndexPage`
* `BookPage`
* categories
* tags
* cover images

Because `book` depends on both:

```
cms_taxonomy
core_images
```

you are now ready for it.

---

If you'd like, I can also show you **a quick dependency map of all apps in your old project** so we avoid any surprises while importing them step-by-step.
