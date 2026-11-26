[English](README.md) | 日本語

# SQLite(する のライブラリ)

日本語のプログラミング言語「[する](https://github.com/suru-lang/suru)」から SQLite のデータベースを使うライブラリです。libsqlite3 を外部の関数として呼びます。

## 入れ方

[kumu](https://github.com/suru-lang/suru/blob/main/README_jp.md#プロジェクトを組むkumu) のプロジェクトで:

```sh
kumu 追加 github.com/suru-lang/sqlite
```

データベースの層 [「蔵」](https://github.com/suru-lang/db)(suru-lang/db)も依存として一緒に入ります。システムに libsqlite3 が要ります(組.suru の `外部依存`。無ければ kumu が入れ方を出して止まります)。

- macOS: `brew install sqlite` して `export PKG_CONFIG_PATH="$(brew --prefix sqlite)/lib/pkgconfig"`(macOS にも libsqlite3 は入っていますが、kumu が確かめに使う pkg-config の .pc ファイルがありません)
- Debian・Ubuntu: `sudo apt install libsqlite3-dev`
- Fedora: `sudo dnf install sqlite-devel`
- Windows(MSYS2 の UCRT64): `pacman -S mingw-w64-ucrt-x86_64-sqlite3`。UCRT64 のシェルで clang・lld・pkgconf(`pacman -S mingw-w64-ucrt-x86_64-clang mingw-w64-ucrt-x86_64-lld mingw-w64-ucrt-x86_64-pkgconf`)と、する 0.8.0 以上(Windows に対応した最初の版)を使います

する 本体は 0.9.0 以上が要ります(ライブラリが手順を「以下の手順です … 以上です」で、C の関数を「以下の外部の入出力動作です … 以上です」で書くため)。

## 使い方

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

名前はどれも `SQL` か `SQLite` で始まります。`「SQLite」 を 読み込む` なら `SQLite内のSQLite開封する` のように書きます。例は [例/店.suru](例/店.suru) です。「蔵」を通して使う例は [例/層.suru](例/層.suru) です。

## 決めたこと

1. **開く・閉じる。** `道 を SQLite開封する` はファイルのデータベースを開き(なければ作る)、接続(`SQLite接続`)を結果にします。`「:memory:」` ならメモリの中だけのデータベースです。`db を SQLite閉鎖する` で閉じます。どれも動作です。
2. **値は選択肢のある型 `SQL値`。** `SQL無`(NULL)、`42 の SQL数`(整数なら INTEGER、小数なら REAL として入れる)、`「a」 の SQL文字列`(TEXT)、`b の SQLバイト列`(BLOB)です。読んだ値もこの型で、パターンマッチで取り出せます。`v を SQL値文字列にする` は表示のための文字列(`SQL無` は「NULL」)です。
3. **実行する・問い合わせる。** `db に 文 を SQLite実行する` は結果の行のない SQL を実行して、変わった行の数を結果にします(「;」で区切って何文でも書けます)。`db に 文 を SQLite問合する` は結果の行のリストで、1行は `SQL値` のリスト(列の順)です。`db に 文 を SQLite列名取得する` は列の名前のリスト、`db を SQLite最終行番号取得する` は最後に入れた行の rowid です。トランザクションは `「BEGIN」`・`「COMMIT」` を SQLite実行する で書きます。
4. **値は「?」に入れる。** `SQLite差込実行する`・`SQLite差込問合する` は、文の「?」に値群(`SQL値` のリスト)を前から順に入れます。SQLite が値として受け取るので、文字列に引用符が入っていても SQL を壊しません。人の入力を SQL に使うときは、文字列を `+` でつながずに、こちらを使います。
5. **失敗は実行時エラー。** 「SQLite で『SELEC 1』を実行するのに失敗しました(near "SELEC": syntax error)」のように、SQLite の理由を付けた実行時エラーになります(`試行する` で受け止められます)。失敗しても、準備した文は片付けます。
6. **たくさんの行も読める。** 行は手順の終わりでの繰り返しで集めるので、何千行でも深くなりません。
7. **「蔵」の駆動 `SQLite駆動`。** `先 を SQLite駆動 で DB開封する`(先 はファイルの道か `「:memory:」`)で、データベースの層「蔵」の関数(`DB実行する`・`DB差込問合する`・`DB行問合する`・`DB取引する` など)から SQLite を使えます。駆動の名前を替えれば、同じプログラムでほかのデータベースを使えます。`DB真偽` は 1 と 0 で入れ、読むと `DB数` になります(`DB真偽取出する` で真偽に戻せます)。値のない `DB実行する` は、`SQLite実行する` と同じく「;」で区切って何文でも書けます。

## 試験

```sh
kumu 試験
kumu 点検
```

CI では Linux・macOS・Windows(MSYS2)で動かします。

## 今後の候補

- 準備した文を何度も使う(同じ INSERT をたくさん行うときに速くする)
- 「?」のほかに「:名前」の名前つきの差し込み
- 大きな結果を1行ずつ読む(全部をリストにしないで畳み込む)

## ライセンス

[MIT](LICENSE)
