# デプロイ

## プラットフォーム
GitHub Pages

## URL
https://mor-nhatlq.github.io/development-cycle/

## ソース
- リポジトリ: https://github.com/mor-nhatlq/development-cycle
- ブランチ: `main`
- パス: `/`（ルート）
- エントリ: `index.html`（`development-cycle.html`からリネーム）

## ローカルステージングディレクトリ
`/Users/dangtuanphong/Desktop/AI-Workflow/.deploy/development-cycle/`

オリジナルソースの`development-cycle.html`はワークスペースルートに配置されております。デプロイ用のコピーは`index.html`としてコミットされ、GitHub Pagesがリポジトリのルート URLで配信いたします。

## デプロイコマンド（以降の更新時）
```bash
cp development-cycle.html .deploy/development-cycle/index.html
cd .deploy/development-cycle
git add index.html
git commit -m "update: <message>"
git push
```
GitHub Pagesは約1分以内に自動的に再ビルドいたします。

## 環境変数
なし。

## カスタムドメイン
未設定です。追加する場合: `gh api -X PUT repos/mor-nhatlq/development-cycle/pages -f cname=<domain>`を実行し、`mor-nhatlq.github.io`を指すCNAME DNSレコードを追加いたします。

## ロールバック
```bash
cd .deploy/development-cycle
git revert HEAD
git push
```
または以前のコミットへリセットしてフォースプッシュ（破壊的操作 — 実行前に必ず確認をお願いいたします）。

## Pages無効化
```bash
gh api -X DELETE repos/mor-nhatlq/development-cycle/pages
```
