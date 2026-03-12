Perfect. Let’s do **Phase 1: import `cms_taxonomy`** into `dmlibrary_local` as a small, safe first step.

Why this first? Because in the old repo, `book` depends on `cms_taxonomy.BookCategory`, so bringing taxonomy in first gives you the snippet/category foundation before importing content apps.

## What `cms_taxonomy` contains

In the old repo, `cms_taxonomy/models.py` defines reusable snippet-style category models used by content apps, including book categories. That is why it is the right first import target.

## Phase 1 checklist

### Step 0: make a safety commit in `dmlibrary_local`

In your new repo:

```bash
git status
git add .
git commit -m "Checkpoint before importing cms_taxonomy"
```

### Step 1: copy the app directory

From the old repo, copy this folder into the new repo root:

```text
cms_taxonomy/
```

Also check whether the app contains:

* `models.py`
* `migrations/`
* `apps.py`
* `admin.py`
* templates, if any

The old repo does include a `cms_taxonomy` app directory.

### Step 2: add the app to `INSTALLED_APPS`

In `dmlibrary_local/mysite/settings/base.py`, add:

```python
"cms_taxonomy",
```

Put it with your custom apps, for example after `search` or near the other local apps.

In the old repo, `cms_taxonomy` is included in `INSTALLED_APPS`, which confirms it is meant to be a normal Django app in the project.

### Step 3: check imports inside `cms_taxonomy/models.py`

Open `cms_taxonomy/models.py` and look for imports that may require other old apps.

You want this phase to stay isolated. If you see imports from apps like:

* `book`
* `photo`
* `blog`
* `core_images`

then stop and paste that file here before going further.

If it only uses Django/Wagtail/snippet-related imports, you can proceed.

### Step 4: build and run migrations

From the new repo:

```bash
docker compose -f docker-compose.dev.yml exec web python manage.py makemigrations cms_taxonomy
docker compose -f docker-compose.dev.yml exec web python manage.py migrate
```

If you copied the old `migrations/` folder too, you may be able to skip `makemigrations` and just run:

```bash
docker compose -f docker-compose.dev.yml exec web python manage.py migrate
```

### Step 5: verify in Wagtail admin

Open:

```text
http://localhost:8000/admin
```

Look for the snippet/category entry that came from `cms_taxonomy`.

Because this app is intended to provide reusable taxonomy models, the expected result is that category models become available in admin/snippets after migration.

### Step 6: test from Django shell

Run:

```bash
docker compose -f docker-compose.dev.yml exec web python manage.py shell
```

Then try something like:

```python
from cms_taxonomy.models import *
```

If that imports cleanly, the app is wired in correctly.

### Step 7: commit immediately

Once it works:

```bash
git add .
git commit -m "Import cms_taxonomy app"
```

---

## What not to copy yet

Do **not** copy these yet:

* `book`
* `photo`
* `blog`
* `core`
* `cms_migration`

Those come later. In particular, `book` should wait until after `cms_taxonomy` and `core_images`, because the old `book` app depends on both category and custom image pieces.

## If migration fails

The most likely issues are:

* missing dependency import inside `cms_taxonomy/models.py`
* old migration files referring to apps not yet imported
* Wagtail version differences between the old repo and new repo

If that happens, send me these two files from the old repo and I’ll adapt them for the new repo:

* `cms_taxonomy/models.py`
* `cms_taxonomy/migrations/0001_initial.py`

## After this phase

The next phase should be:

1. `core_images`
2. `book`

That order matches the dependency shape from the old repo. `book` references both taxonomy and the custom image model.

If you paste `cms_taxonomy/models.py` after copying it, I’ll check it before you run migrations.
