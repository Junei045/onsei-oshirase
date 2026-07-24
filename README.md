# 豊田連合 音声のお知らせ

住民向けの再生ページと、役員向けの管理画面です。GitHub Pages + Firebase で動きます。

## ファイル

| ファイル | 役割 |
|---|---|
| `index.html` | 再生ページ（住民向け・公開） |
| `admin.html` | 管理画面（役員向け・Googleログイン必須） |
| `firestore.rules` | Firestore のセキュリティルール |
| `storage.rules` | Storage のセキュリティルール |

## 手順1：まず見た目を確認する（Firebase 不要）

`index.html` をブラウザで開くだけです。デモ表示になり、サンプルのお知らせが3件並びます。
デモの音声は**無音**です。再生ボタンのリングが進むこと、同時に2つ鳴らないこと、
文字が読みやすいことを確認してください。

ここで直したいところがあれば、Firebase を触る前に直します。

## 手順2：Firebase を用意する

1. Firebase コンソールで新規プロジェクトを作成
2. **Authentication** → ログイン方法 → Google を有効化
3. **Firestore Database** を作成（本番モード）
4. **Storage** を作成
5. 「プロジェクトの設定」→「マイアプリ」→ ウェブアプリを追加し、`firebaseConfig` をコピー
6. コピーした内容を `index.html` と `admin.html` の冒頭にある `firebaseConfig` に貼り付け（**両方**）

## 手順3：ルールを反映する

`firestore.rules` と `storage.rules` の中身を、Firebase コンソールの各「ルール」タブに貼り付けて公開します。

## 手順4：管理者を登録する

Firestore に `admins` コレクションを作り、**管理者にしたい人の UID をドキュメントIDにした空のドキュメント**を追加します。
UID は Authentication のユーザー一覧で確認できます。

一度 `admin.html` でログインを試すとユーザーが作られるので、UID をコピーして登録し、再度ログインしてください。

## 手順5：APIキーを制限する

Google Cloud コンソール → 認証情報 → 該当APIキー → 「アプリケーションの制限」で
HTTP リファラーを `https://junei045.github.io/*` に限定します。
現況届・bippi と同じ手当てです。

## データの形

`broadcasts` コレクション:

```
category      "urgent" | "event" | "info"
title         見出し
body          本文（音声と同じ内容）
audioPath     Storage 上のパス
audioUrl      再生用URL
durationSec   長さ（秒・自動取得）
publishedAt   公開日時
expiresAt     掲載終了日時（null可）
published     true / false
createdBy     操作者UID
createdByName 操作者名
createdAt     登録日時
```

`broadcast_logs` コレクション: 誰がいつ何をしたかの記録（追記のみ・変更削除不可）

## 運用上の決めごと

- **緊急**を選ぶと、行政発表を確認したかのチェック欄が出ます。チェックしないと公開できません
- 緊急のお知らせには**掲載終了日時を必ず入れて**ください。古い警報が残り続けるのを防ぎます
- 本文は音声と同じ内容にしてください。音声を再生できない方はこの文章だけを読みます

## 次のステップ

1. TTS（合成音声）生成を管理画面に組み込む
2. 役員会での運用ルール承認
3. LINE公式アカウントからの一斉配信連携
