# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🔄 開発の上昇サイクル / 🔒 セキュリティ＆コスト厳守ルール

このプロジェクトの開発フロー・セキュリティ方針・コスト方針の正式な定義は
[`docs/manual/development_style.md`](docs/manual/development_style.md) を参照すること。
修正完了時は同ファイル配下(`docs/manual/`)へ技術履歴サマリーを自動蓄積する。

特に以下は毎セッション厳守:
- **個人情報・認証情報・クライアント情報をコードにハードコード/読込しない**
- **コスト・トークン節約のため、コード変更は常に最小限かつ簡潔に。不要な外部ライブラリを勝手に追加しない**

## Project Overview

**Google Cloud Digital Leader 対策クイズ**は、Google Cloud認定資格「Cloud Digital Leader」の試験対策を行う単一HTMLファイルのWebアプリです(非公式・学習用)。試験ガイドに沿った4セクション構成で全50問を収録しています。

本番公開URL: **https://yoshioide2301-dev.github.io/GcpExamApp/**
(GitHub Pages / リポジトリ `GcpExamApp` は Public)

## 🚀 起動・開発コマンド

- ローカル起動: `node server.js` → `http://localhost:8000` でアクセス
  - `server.js` は依存パッケージ不要の素朴な静的ファイルサーバー(`http`/`fs`のみ使用)。`npm install` は不要。
- ビルドステップは存在しない。`index.html` を直接編集し、保存すればそのまま動作に反映される。
- デプロイ: `main` ブランチに push すると GitHub Pages が自動再ビルド・再公開する(手動デプロイ操作は不要)。
  ```
  git add .
  git commit -m "..."
  git push
  ```

## 🎨 コードスタイル・デザインルール

- **単一ファイル構成が絶対ルール**: HTML/CSS/JS は全て `index.html` 一枚に `<style>`/`<script>` インラインで記述する。別ファイルへの分割(`.css`/`.js`切り出し)はしないこと。
- **配色は Google Material Design 準拠のライトテーマ**(DroneExamAppとは異なりダークテーマではない)。`:root` で定義された以下のCSS変数を踏襲すること:
  - `--blue:#1a73e8` / `--green:#188038` / `--red:#d93025` / `--yellow:#f9ab00`(Googleブランドカラー)
  - `--bg:#f6f8fc`(背景) / `--surface:#ffffff`(カード面) / `--text:#202124` / `--text-sub:#5f6368`
  - 角丸は `--radius:16px`、影は `--shadow` 変数を共通利用する。
- 画面遷移はSPA的なルーティングライブラリを使わず、`#app` 要素の中身を `renderMenu()` / `renderQuiz()` / `renderResult()` が丸ごと差し替える方式。新しい画面を追加する場合もこのパターンに合わせる。
- フッターに「非公式・学習用」であることを明示するクレジット表記があるため、削除しないこと。

## 💾 データ構造・プロジェクト構造

```
GcpApp/
├── index.html   # アプリ本体(HTML+CSS+JS全部入り、デプロイ対象)
└── server.js    # ローカル確認用の簡易静的サーバー
```

- **`SECTION_META`**: 4セクションのタイトル等のメタ情報を持つオブジェクト。
  1. デジタル トランスフォーメーション
  2. イノベーション(データ・AI/ML)
  3. インフラとアプリケーションの最新化
  4. セキュリティ、オペレーション、料金、サポート
- **`QUESTIONS`**: 全50問の設問データ配列。各要素は以下の形式:
  ```js
  { id:"s1-01", section:1, q:"...", choices:[...], correct:0, explain:"..." }
  ```
  - `id` の接頭辞 `sN-` の `N` は所属セクション番号(`section`フィールドと対応)。
- **弱点克服モード用 localStorage キー**: `cdl_quiz_wrong_ids_v1`(定数名 `STORAGE_KEY`)
  - 誤答した設問の `id` を JSON 配列として保存する。次回起動時にこのキーを読み込み、弱点だけを抽出した「弱点克服モード」の復習セッションを構成する(`renderQuiz()` 内の `isWeakMode` 分岐)。
  - localStorage が使用できない環境では例外を握りつぶし、記憶をスキップする設計(`try/catch`)になっているため、この挙動を壊さないこと。
