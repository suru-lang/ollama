# Ollama(する のライブラリ)

日本語のプログラミング言語「[する](https://github.com/suru-lang/suru)」から [Ollama](https://ollama.com) の API を使うライブラリです。チャット(1文字ずつ流す・まとめて受け取る)、道具(tools)の呼び出し、考えるモデルの考え、モデルの一覧と詳細、Ollama の web 検索 API を使えます。中身は全部 する で書いてあり、[suru-lang/http](https://github.com/suru-lang/http) のクライアントで Ollama に聞きます。

[suru-lang/ohanashi-ollama](https://github.com/suru-lang/ohanashi-ollama)(ブラウザで Ollama とおしゃべりする Web アプリ)の Ollama とのやりとりを切り出したものです(suru-lang/ohanashi-ollama#31)。

## 入れ方

[kumu](https://github.com/suru-lang/suru#プロジェクトを組むkumu) のプロジェクトで:

```sh
kumu 追加 github.com/suru-lang/ollama
```

依存の suru-lang/http@0.4.0 の HTTPS のために、システムに OpenSSL が要ります(Debian・Ubuntu なら `sudo apt install libssl-dev`、macOS なら `brew install openssl@3` して `export PKG_CONFIG_PATH="$(brew --prefix openssl@3)/lib/pkgconfig"`)。手元の Ollama(`http://localhost:11434`)だけに聞くときも、作るときに要ります。

## 使い方

```
「標準」 を 読み込んで 使う
「Ollama」 を 読み込んで 使う

# 届いた断片を、そのまま表示する
数 を Ollama通知 で 表示受けする は 数 の 動作
n を （s という Ollama断片） で 表示受けする とは 手順
  s を 続表示する
  n + 1 を 結果にする
n を x で 表示受けする とは n を 結果にする

問答する は （数 と 文字列） の 成否 の 動作
問答する とは 入出力手順
  w とは 「http://localhost:11434」 を Ollama窓口にする
  依頼 とは 「llama3.2」 で （（「user」 で 「空はなぜ青いの?」 を Ollama発言にする） を 空 に 挿入する） を Ollama依頼にする
  # 流す
  個数 とは w で 依頼 を 0 から 表示受けすること に Ollamaチャット流した 中身
  （改行 + （個数 を 文字列化する） + 「 個の断片」） を 表示する
  # まとめて受け取る
  答 とは w で 依頼 を Ollamaチャットした 中身
  答 の 発言 の 内容 を 表示する
  0 の 成功 を 結果にする
```

名前はどれも `Ollama` で始まります。`「Ollama」 を 読み込む` なら `Ollama内のOllama窓口にする` のように書きます。例は [例/一問.suru](例/一問.suru)(問いを1つ送って、答えを少しずつ表示する)です。

### 窓口

| 書き方 | 結果 |
|---|---|
| `基点 を Ollama窓口にする` | 窓口(`Ollama窓口`: 基点 と 鍵)。基点は `http://localhost:11434` など(終わりの「/」は除く)。鍵は「」 |
| `w に 鍵 を Ollama鍵設定する` | 鍵を入れた窓口。どの要求にも `Authorization: Bearer 鍵` を付ける(ollama.com の API や web 検索のとき) |

### 発言と依頼

| 書き方 | 結果 |
|---|---|
| `役 で 内容 を Ollama発言にする` | 発言(`Ollama発言`: 役・内容・考え・道具呼出群・道具名)。役は `system`・`user`・`assistant`・`tool` |
| `名 で 返答 を Ollama道具返答にする` | 道具の結果を返す発言(役 `tool`、道具名 名) |
| `モデル で 発言群 を Ollama依頼にする` | 依頼(`Ollama依頼`: モデル・発言群・道具群・選択群・形式・考える)。道具・options・format・think はなし |
| `依頼 に 名 を 値 で Ollama選択追加する` | options に1つ足した依頼(温度なら `「temperature」 を （0.5 の JSON数） で`) |
| `名 で 説明 を 形 から Ollama道具定義にする` | tools に入れる道具の定義の JSON(形は引数の JSON スキーマ) |
| `呼出 を Ollama道具名取得する` / `Ollama道具引数取得する` | 道具の呼び出し(tool_calls の1つ)の名前 / 引数の JSON |
| `発言 を Ollama発言JSONにする` | 発言の JSON(考え・道具呼出群・道具名は、あるときだけ付ける) |
| `依頼 を 流す で Ollama要求本文にする` | `/api/chat` に送る本文(JSON の文字列)。流す は 真偽 |

道具・format・think は、組の書きかえで付けます: `道具群 は 群 の 依頼`、`形式 は 形 の 依頼`(`「json」 の JSON文字列` か JSON スキーマ)、`考える は （真 の JSON真偽） の 依頼`。

### チャット

| 書き方 | 結果 |
|---|---|
| `w で 依頼 を 初め から 受け方 に Ollamaチャット流する` | 届いた1行ごとに `Ollama通知` にして `受け方` を呼び、状態をたたみ込んだ最後の状態 |
| `w で 依頼 を Ollamaチャットする` | 流さずに、答え(`Ollama答え`: 発言・語数・秒・送語数) |
| `行 を Ollama通知にする` | `/api/chat` から届いた1行の中身 |

`受け方` は `（状態 を 通知 で 受け取って 状態 の 動作 を 返すこと）` です。`Ollama通知` は次のどれかです。

| 通知 | 中身 |
|---|---|
| `s の Ollama断片` | 答えの続き |
| `s の Ollama考え` | 考えている途中の文(考えるモデルのとき) |
| `群 の Ollama道具呼出` | モデルが道具を呼んだ(tool_calls の中身の JSON のリスト) |
| `語数 と 秒 と 送語数 の Ollama完了` | 最後の行。語数は答えのトークンの数、秒は答えを作るのにかかった秒、送語数は送った発言のトークンの数 |
| `理由 の Ollama誤り` | 流している途中に Ollama が知らせた失敗 |
| `Ollama無言` | 何も入っていない行(読めない行も) |

### モデル

| 書き方 | 結果 |
|---|---|
| `w を Ollamaモデル一覧取得する` | 入っているモデルの名前のリスト(`GET /api/tags`) |
| `w を Ollamaモデル群取得する` | 入っているモデル(`Ollamaモデル`: 名前・識別(digest)・大きさ(バイト)・族・パラメータ(「7.6B」など)・量子化)のリスト |
| `w で 名前 を Ollamaモデル詳細取得する` | `Ollamaモデル詳細`: 能力群(`completion`・`tools`・`thinking`・`vision`・`embedding` など)と文脈長(`POST /api/show`) |
| `文 を Ollamaパラメータ数にする` | 「7.6B」「494M」「1.2T」を 10 億を単位にした数(7.6・0.494・1200)。読めなければ 0 |
| `w を Ollama版取得する` | Ollama の版(`GET /api/version`) |

### ウェブ検索

| 書き方 | 結果 |
|---|---|
| `w に 問い を 件数 で Ollamaウェブ検索する` | `POST 基点/api/web_search`。`Ollama検索結果`(題・URL・中身)のリスト |

窓口は `「https://ollama.com」 を Ollama窓口にして 鍵 で Ollama鍵設定する` のように、ollama.com と鍵で作ります(鍵は https://ollama.com/settings/keys で作ります)。

### JSON を見る補助

| 書き方 | 結果 |
|---|---|
| `v から 名前 を Ollama文字値取得する` / `Ollama数値取得する` | JSON の項目の文字列(なければ「」)/ 数(なければ 0) |

## 決めたこと

### 窓口と失敗

1. **動作の結果はどれも成否。** つながらない、Ollama に断られた、答えが JSON でないときは、理由の `失敗` を結果にします(実行時エラーにはしません)。「…した 中身」で受け取れます。
2. **理由の文。** 答えが `{"error":"…"}` なら「Ollama: model 'x' not found」のように Ollama の文だけにします。401・403 は「Ollama に断られました(401: …)。鍵を確かめてください」、ほかの状態は「宛先 の状態が 404 でした: 本文」、つながらないときは「宛先 に聞けませんでした: …」です。鍵は理由に入れません。
3. **基点は窓口に持つ。** 道(`/api/chat` など)は基点の後ろにつなぎます。Ollama を別の主機や口番で動かしているときは、基点を変えます。環境変数(`OLLAMA_HOST` など)はライブラリでは読まず、使う側で決めます。

### 発言と依頼

1. **ライブラリ自前の型にする。** 発言は `Ollama発言`、送るものは `Ollama依頼` です。使う側のデータ(会話の記録など)からは、使う側で詰め替えます。
2. **あるものだけを送る。** 発言の `thinking`・`tool_calls`・`tool_name`、依頼の `tools`・`options`・`format`・`think` は、空(か `JSON無`)なら本文に入れません。Ollama の既定のままにするためです(温度を送らなければ、モデルの既定の温度)。
3. **道具の定義は JSON のまま。** 引数の形(JSON スキーマ)はいろいろなので、組にせずに JSON で渡します。`Ollama道具定義にする` は `{"type":"function","function":{…}}` の形にするだけです。

### チャット

1. **流すときは状態をたたみ込む。** 届いた行ごとに `Ollama通知` にして、渡された動作を呼びます(suru-lang/http の `逐次送付する` と同じ形)。画面に送る、文をつなぐ、道具の呼び出しを集める、などは使う側の動作で行います。
2. **途中の失敗は通知で、はじめの失敗は失敗で。** 流しはじめる前の失敗(ないモデルで 404 など)は `失敗` です。流している途中に Ollama が `{"error":…}` を送ったときは、`Ollama誤り` の通知として届きます。
3. **道具を呼ばなくなるまで回すのは使う側で。** 道具の呼び出しが届いたら、使う側が道具を動かし、`assistant` の発言(道具呼出群 付き)と `Ollama道具返答にする` の発言を足して、もう一度チャットします。回す回数の上限や、画面への見せ方は使う側で決めます。
4. **秒は答えを作るのにかかった秒。** `eval_duration`(ナノ秒)を秒にしたものです。読み込みや、送った発言を読む時間は入りません。

### モデル

1. **文脈長は `〜.context_length` を探す。** `/api/show` の `model_info` の項目の名前は族ごとに違う(`llama.context_length`・`qwen3.context_length` など)ので、`.context_length` で終わる最初の項目を使います。
2. **capabilities がなければ空。** 古い Ollama は `capabilities` を返さないので、能力群は空にします(使えないとは限りません)。

### 作らなかったもの

1. **`/api/generate`・`/api/embed`・`/api/pull` などは、まだ作らない。** ohanashi-ollama が使うものから作りました。

## 試験

```sh
kumu 試験
kumu 点検
```

試験は [試験/偽Ollama.suru](試験/偽Ollama.suru) の偽の Ollama(する で書いたもの)を相手にします。流す・流さないチャット、考え、道具の呼び出し、途中の誤り、状態ごとの理由、モデルの一覧と詳細、web 検索を確かめます。

## 今後の候補

- `/api/generate`(1つの問いに答える)と `/api/embed`(埋め込み)
- `/api/pull`(進み具合を流す)・`/api/ps`(読み込んでいるモデル)
- 画像を発言に付ける(`images`)
- 流している途中で止める

## ライセンス

[MIT](LICENSE)
