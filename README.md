# 豊田連合 音声のお知らせ

住民向けの再生ページと、役員向けの管理画面です。
**GitHub Pages + Firebase Firestore** で動きます。音声ファイルはこのリポジトリの `audio/` に置きます。

Firebase の Storage は使いません（従量課金プランへの登録が必要になるため）。
そのため**クレジットカードの登録は不要**で、無料の Spark プランのまま運用できます。

## ファイル構成

```
index.html          再生ページ（住民向け・公開）
admin.html          管理画面（役員向け・Googleログイン必須）
firestore.rules     Firestore のセキュリティルール
audio/              音声ファイル置き場
  README.md         置き方の決まりごと
```

## 手順1：見た目の確認（Firebase 不要）

公開URL <https://junei045.github.io/onsei-oshirase/> を開くとデモ表示になります。
サンプルのお知らせが3件並びます（音声は無音）。

## 手順2：Firebase を用意する

1. Firebase コンソールで新規プロジェクトを作成（表示名は「音声配信システム」、プロジェクトIDは `onsei-oshirase`）
2. Google Analytics は不要なのでオフで構いません
3. **Firestore Database** を作成 → **本番モード** → ロケーション **asia-northeast1（東京）**
   - ロケーションは後から変更できません
4. **Authentication** → ログイン方法 → **Google** を有効化
5. Authentication →「設定」→「承認済みドメイン」に `junei045.github.io` を追加
6. 「プロジェクトの設定」→「マイアプリ」→ ウェブアプリを追加し `firebaseConfig` をコピー
7. コピーした内容を `index.html` と `admin.html` の冒頭にある `firebaseConfig` に貼り付け（**両方**）

Storage は作成不要です。

## 手順3：ルールを反映する

`firestore.rules` の中身を Firebase コンソールの Firestore →「ルール」タブに貼り付けて公開します。

## 手順4：複合インデックスを作る

再生ページは「公開中のものを新しい順に」取得するため、Firestore が複合インデックスを要求します。

作らずに開くと一覧が出ず、ブラウザのコンソール（F12）にインデックス作成用のURLを含むエラーが出ます。
**そのURLをクリックすれば自動で作成されます。** 数分で有効になります。

## 手順5：管理者を登録する

1. `admin.html` でログインを一度試す（「権限がありません」と出るのが正常）
2. Authentication のユーザー一覧で自分の **UID** をコピー
3. Firestore に `admins` コレクションを作り、**UID をドキュメントIDにした空のドキュメント**を追加
4. もう一度ログイン

## 手順6：APIキーを制限する

Google Cloud コンソール → 認証情報 → 該当APIキー → 「アプリケーションの制限」で
HTTPリファラーを `https://junei045.github.io/*` に限定します。

**すべて動作確認できてから**行ってください。先に制限すると切り分けが難しくなります。

## 日々の使い方

1. 音声を録音する
2. `audio/` フォルダに **Add file → Upload files** でアップロードし Commit
3. 1〜2分待つ
4. `admin.html` を開き「再読み込み」→ ファイルを選ぶ
5. 見出しと本文を入れて「公開する」

## データの形

`broadcasts` コレクション:

| 項目 | 内容 |
|---|---|
| `category` | `urgent` / `event` / `info` |
| `title` | 見出し |
| `body` | 本文（音声と同じ内容） |
| `audioPath` | `audio/ファイル名` |
| `audioUrl` | 同上（再生に使用） |
| `durationSec` | 長さ（秒・自動取得） |
| `publishedAt` | 公開日時 |
| `expiresAt` | 掲載終了日時（null可） |
| `published` | true / false |
| `createdBy` / `createdByName` | 操作者 |
| `createdAt` | 登録日時 |

`broadcast_logs` コレクション: 誰がいつ何をしたかの記録（追記のみ・変更削除不可）

## 運用上の決めごと

- **緊急**を選ぶと、行政発表を確認したかのチェック欄が出ます。チェックしないと公開できません
- 緊急のお知らせには**掲載終了日時を必ず入れて**ください。古い警報が残り続けるのを防ぎます
- 本文は音声と同じ内容にしてください。音声を再生できない方はこの文章だけを読みます
- 災害時の正式な情報は横浜市・栄区の発表です。この仕組みは地域からの補足に限ります

## 引き継ぎのために

この仕組みに必要なものは次の3つだけです。課金契約はありません。

1. GitHub リポジトリの管理権限
2. Firebase プロジェクトのオーナー権限
3. Firestore の `admins` コレクションへの UID 登録

役員交代時は、新任の方の GitHub アカウントと Google アカウントを上記に追加してください。

## 次のステップ

1. TTS（合成音声）生成の組み込み
2. 役員会での運用ルール承認
3. LINE公式アカウントからの一斉配信連携
