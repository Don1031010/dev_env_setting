これは **まだマイグレーションが走っていない** か、**SQLite から Postgres に切り替えた後に新しいDBが空のまま** という意味です 👍

`wagtailcore_site` は Wagtail の基本テーブルなので、これがないのは

* Postgres DB は作られた
* でも Django/Wagtail のテーブル作成がまだ

という状態です。

## やること

まずこれを実行してください。

```bash
docker compose -f docker-compose.dev.yml exec web python manage.py migrate
```

これで Django と Wagtail のテーブルが作られます。

---

## そのあと

Wagtail では初回にサイトやページが必要なので、必要ならこちらも実行します。

```bash
docker compose -f docker-compose.dev.yml exec web python manage.py createsuperuser
```

その後、もう一度開いてください。

```text
http://localhost:8000
```

管理画面は:

```text
http://localhost:8000/admin
```

---

## もし `migrate` でエラーが出たら

そのまま結果を貼ってください。
ただ、今のエラー内容だと、かなり素直に **`migrate` 未実行** の可能性が高いです。

## よくある流れ

SQLite から Postgres に切り替えた直後は、以前の `db.sqlite3` に入っていたテーブルは **Postgres には存在しません**。
なので、Postgres 側で改めて:

1. `migrate`
2. `createsuperuser`

が必要です。

実行後、かなり前に進むはずです。
