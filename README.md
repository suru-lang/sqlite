English | [日本語](README_jp.md)

# SQLite (a library for Suru)

A library for using SQLite databases from [Suru (「する」)](https://github.com/suru-lang/suru), a programming language in Japanese. It calls libsqlite3 as foreign functions.

## Installing

In a [kumu](https://github.com/suru-lang/suru#building-projects-kumu) project:

```sh
kumu 追加 github.com/suru-lang/sqlite
```

libsqlite3 is required on the system (the `外部依存` in 組.suru; if it is missing, kumu shows how to install it and stops).

- macOS: `brew install sqlite`, then `export PKG_CONFIG_PATH="$(brew --prefix sqlite)/lib/pkgconfig"` (macOS ships libsqlite3 too, but not the pkg-config .pc file kumu uses to check for it)
- Debian, Ubuntu: `sudo apt install libsqlite3-dev`
- Fedora: `sudo dnf install sqlite-devel`

Suru itself must be a version that includes external dependencies (suru-lang/suru#88) and negative addresses (suru-lang/suru#85).

## Usage

```
「SQLite」 を 読み込んで 使う

主処理する とは 以下の入出力手順です
  db とは 「店.db」 を SQLite開封した 結果
  db に 「CREATE TABLE IF NOT EXISTS 品 (名前 TEXT, 値段 REAL)」 を SQLite実行する
  db に 「INSERT INTO 品 VALUES (?, ?)」 を （（「りんご」 の SQL文字列） と （120 の SQL数）） で SQLite差込実行する
  行群 とは db に 「SELECT 名前 FROM 品 WHERE 値段 < ?」 を （（200 の SQL数） を 空 に 挿入する） で SQLite差込問合した 結果
  行群 を 表示する
  db を SQLite閉鎖する
  以上です

主処理する
```

Every name starts with `SQL` or `SQLite`. With `「SQLite」 を 読み込む`, write them with the prefix, as in `SQLite内のSQLite開封する`. An example is [例/店.suru](例/店.suru).

## Decisions

1. **Opening and closing.** `道 を SQLite開封する` opens a database file (creating it if absent) and gives a connection (`SQLite接続`). `「:memory:」` gives a database only in memory. `db を SQLite閉鎖する` closes it. All of these are actions.
2. **Values have the type with alternatives `SQL値`.** `SQL無` (NULL), `42 の SQL数` (stored as INTEGER if an integer, REAL if a decimal), `「a」 の SQL文字列` (TEXT) and `b の SQLバイト列` (BLOB). Values read back have this type too and can be taken apart with pattern matching. `v を SQL値文字列にする` is a string for display (「NULL」 for `SQL無`).
3. **Executing and querying.** `db に 文 を SQLite実行する` runs SQL without result rows and gives the number of changed rows (any number of statements separated by 「;」). `db に 文 を SQLite問合する` gives the list of result rows, each row a list of `SQL値` (in column order). `db に 文 を SQLite列名取得する` gives the list of column names, and `db を SQLite最終行番号取得する` the rowid of the last inserted row. Transactions are written by running `「BEGIN」` and `「COMMIT」` with SQLite実行する.
4. **Values go into 「?」.** `SQLite差込実行する` and `SQLite差込問合する` put the values (a list of `SQL値`) into the statement's 「?」 in order from the front. SQLite receives them as values, so quotes inside strings can't break the SQL. When using people's input in SQL, use these instead of joining strings with `+`.
5. **Failures are runtime errors.** They are runtime errors with SQLite's reason, like 「SQLite で『SELEC 1』を実行するのに失敗しました(near "SELEC": syntax error)」 ("failed to run 『SELEC 1』 in SQLite (…)") (catchable with `試行する`). Prepared statements are cleaned up even on failure.
6. **Many rows can be read.** Rows are collected by a loop at the end of a procedure, so thousands of rows don't make it deep.

## Tests

```sh
kumu 試験
kumu 点検
```

## Future work

- Look rows up by column name (records of column names and values, or queries returning one row as a 「表」)
- A function wrapping transactions (pass a procedure; ROLLBACK on failure)
- Reuse prepared statements (to speed up running the same INSERT many times)
- Named placeholders 「:名前」 in addition to 「?」
- Read large results one row at a time (fold without making a list of everything)

## License

[MIT](LICENSE)
