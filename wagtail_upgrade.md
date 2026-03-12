Yes — I would upgrade. Since you’re on **7.3rc1**, you’re on a **release candidate**, not the stable final release. Wagtail’s docs show **7.3.1** as the current stable patch release in that line of releases, and the 7.3 release notes mention a **permission vulnerability in the Wagtail admin preview endpoints**, so moving off the RC is the safer choice. ([docs.wagtail.org][1])

For your project, this should be a **small, normal upgrade**: **7.3rc1 → 7.3.1** is within the same minor series, so it’s much lower risk than jumping to a new major version. Still, do it the careful way: back up your DB, change the version pin, rebuild, and run migrations. ([docs.wagtail.org][1])

Use this in `requirements.txt`:

```txt
wagtail==7.3.1
```

Then rebuild and restart your dev stack:

```bash
docker compose -f docker-compose.dev.yml build
docker compose -f docker-compose.dev.yml up -d
docker compose -f docker-compose.dev.yml exec web python manage.py migrate
```

You can also verify the installed version inside the container with:

```bash
docker compose -f docker-compose.dev.yml exec web python -m pip show wagtail
```

If you want a slightly more flexible pin, this is also reasonable:

```txt
wagtail>=7.3.1,<7.4
```

But for now, since you’re stabilizing the environment, I’d keep it exact at `7.3.1`.

One more practical note: after upgrading, check the official **7.3.1 release notes** and test your admin, page editing, search, and any custom Wagtail panels or hooks you added. The release notes page for 7.3.1 is available in the Wagtail docs, and the release process page also shows that 7.3.2 is only “in development,” so **7.3.1** is the right stable target today. ([docs.wagtail.org][1])

So the short recommendation is: **yes, upgrade to 7.3.1 now**.

[1]: https://docs.wagtail.org/en/stable/releases/index.html?utm_source=chatgpt.com "Wagtail 7.3.1 release notes"
