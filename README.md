# SQLite(する のライブラリ)

日本語のプログラミング言語「[する](https://github.com/suru-lang/suru)」から SQLite のデータベースを使うライブラリです。libsqlite3 を外部の関数として呼びます。

## 入れ方

[kumu](https://github.com/suru-lang/suru#プロジェクトを組むkumu) のプロジェクトで:

```sh
kumu 追加 github.com/suru-lang/sqlite
```

システムに libsqlite3 が要ります(組.suru の `外部依存`。無ければ kumu が入れ方を出して止まります)。

- macOS: `brew install sqlite` して `export PKG_CONFIG_PATH="$(brew --prefix sqlite)/lib/pkgconfig"`(macOS にも libsqlite3 は入っていますが、kumu が確かめに使う pkg-config の .pc ファイルがありません)
- Debian・Ubuntu: `sudo apt install libsqlite3-dev`
- Fedora: `sudo dnf install sqlite-devel`

する 本体は、外部依存(suru-lang/suru#88)と負の数の番地(suru-lang/suru#85)が入った版が要ります。

## 使い方

```
「SQLite」 を 読み込んで 使う

主処理する とは 入出力手順
  db とは 「店.db」 を SQLite開封した 結果
  db に 「CREATE TABLE IF NOT EXISTS 品 (名前 TEXT, 値段 REAL)」 を SQLite実行する
  db に 「INSERT INTO 品 VALUES (?, ?)」 を （（「りんご」 の SQL文字列） と （120 の SQL数）） で SQLite差込実行する
  行群 とは db に 「SELECT 名前 FROM 品 WHERE 値段 < ?」 を （（200 の SQL数） を 空 に 挿入する） で SQLite差込問合した 結果
  行群 を 表示する
  db を SQLite閉鎖する

主処理する
```

名前はどれも `SQL` か `SQLite` で始まります。`「SQLite」 を 読み込む` なら `SQLite内のSQLite開封する` のように書きます。例は [例/店.suru](例/店.suru) です。

## 決めたこと

1. **開く・閉じる。** `道 を SQLite開封する` はファイルのデータベースを開き(なければ作る)、接続(`SQLite接続`)を結果にします。`「:memory:」` ならメモリの中だけのデータベースです。`db を SQLite閉鎖する` で閉じます。どれも動作です。
2. **値は選択肢のある型 `SQL値`。** `SQL無`(NULL)、`42 の SQL数`(整数なら INTEGER、小数なら REAL として入れる)、`「a」 の SQL文字列`(TEXT)、`b の SQLバイト列`(BLOB)です。読んだ値もこの型で、パターンマッチで取り出せます。`v を SQL値文字列にする` は表示のための文字列(`SQL無` は「NULL」)です。
3. **実行する・問い合わせる。** `db に 文 を SQLite実行する` は結果の行のない SQL を実行して、変わった行の数を結果にします(「;」で区切って何文でも書けます)。`db に 文 を SQLite問合する` は結果の行のリストで、1行は `SQL値` のリスト(列の順)です。`db に 文 を SQLite列名取得する` は列の名前のリスト、`db を SQLite最終行番号取得する` は最後に入れた行の rowid です。トランザクションは `「BEGIN」`・`「COMMIT」` を SQLite実行する で書きます。
4. **値は「?」に入れる。** `SQLite差込実行する`・`SQLite差込問合する` は、文の「?」に値群(`SQL値` のリスト)を前から順に入れます。SQLite が値として受け取るので、文字列に引用符が入っていても SQL を壊しません。人の入力を SQL に使うときは、文字列を `+` でつながずに、こちらを使います。
5. **失敗は実行時エラー。** 「SQLite で『SELEC 1』を実行するのに失敗しました(near "SELEC": syntax error)」のように、SQLite の理由を付けた実行時エラーになります(`試行する` で受け止められます)。失敗しても、準備した文は片付けます。
6. **たくさんの行も読める。** 行は手順の終わりでの繰り返しで集めるので、何千行でも深くなりません。

## 試験

```sh
kumu 試験
kumu 点検
```

## 今後の候補

- 行を列の名前で引けるようにする(列名と値の組や、「表」で1行を返す問い合わせ)
- トランザクションを包む関数(手順を渡して、失敗したら ROLLBACK する)
- 準備した文を何度も使う(同じ INSERT をたくさん行うときに速くする)
- 「?」のほかに「:名前」の名前つきの差し込み
- 大きな結果を1行ずつ読む(全部をリストにしないで畳み込む)

## ライセンス

[MIT](LICENSE)
