# 「SQLite」

SQLite for Suru. It calls libsqlite3 through external functions.
Add it as a dependency with `kumu 追加` and write `「SQLite」 を 読み込んで 使う` in your file. Every name starts with `SQL` or `SQLite`.
With `「SQLite」 を 読み込む`, add the `SQLite内の` prefix, like `SQLite内のSQLite開封する`.

A database value is the type `SQL値`, which has these forms:

- `SQL無`: NULL.
- `42 の SQL数`: a number (INTEGER for an integer, REAL for a decimal).
- `「a」 の SQL文字列`: TEXT.
- `b の SQLバイト列`: BLOB.

Every function that touches the database is an action. When it fails, it is a runtime error such as
「SQLite で『SELEC 1』を実行するのに失敗しました(near "SELEC": syntax error)」, which `試行する` can catch.

`SQLite駆動` is the driver for the database layer 「蔵」 (suru-lang/db): `先 を SQLite駆動 で DB開封する` lets you use SQLite through the functions of 「蔵」 (`DB実行する`, `DB差込問合する`, `DB取引する` and so on). `DB真偽` is stored as 1 and 0.

```{toctree}
:maxdepth: 2
:caption: API
:glob:

api/*
```
