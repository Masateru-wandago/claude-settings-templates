# Claude Code 権限設定テンプレート集

Claude Code の `settings.json` に貼り付けて使える、用途別の権限設定テンプレートです。

## テンプレート一覧

| ファイル | 用途 | 安全度 | 作業効率 |
|---|---|---|---|
| `templates/template-readonly.json` | 読み取り専用（調査・レビュー） | ★★★★★ | ★☆☆☆☆ |
| `templates/template-strict.json` | 厳しめ（本番環境近くでの作業） | ★★★★☆ | ★★☆☆☆ |
| `templates/template-infra-balanced.json` | インフラ作業向けバランス型 | ★★★☆☆ | ★★★☆☆ |
| `templates/template-dev-balanced.json` | アプリ開発・雰囲気コーディング向けバランス型 | ★★★☆☆ | ★★★★☆ |
| `templates/template-loose.json` | ゆるめ（作業効率優先） | ★★☆☆☆ | ★★★★★ |

### `_comment` 付きファイルについて（`templates_comment/` フォルダ）

各テンプレートのコメント付き版を `templates_comment/` フォルダに収録しています（例: `templates_comment/template-dev-balanced_comment.json`）。

`_allow_comments` / `_deny_comments` フィールドに、各権限エントリの意図・理由を記載しています。設定の差分を確認したい時・カスタマイズの参考にしたい時に使ってください。

> **注意:** `_allow_comments` / `_deny_comments` は Claude Code が無視する追加フィールドです。`settings.json` に貼り付けてもそのまま動作します。

---

## 各テンプレートの詳細

### `template-readonly.json` — 読み取り専用

**用途:** コードレビュー、障害調査、環境確認など「見るだけ」の作業に。

- 読み取り・参照系コマンドのみ許可（`cat`, `ls`, `grep`, `git log`, `kubectl get` など）
- `Write` / `Edit` は全てブロック
- `terraform apply`, `kubectl apply` など変更系コマンドはブロック

---

### `template-strict.json` — 厳しめ

**用途:** 本番環境に近いインフラ作業、慎重に進めたい時。

- Git 操作と参照系コマンドは許可
- `terraform plan/validate`, `kubectl get/describe/logs` など確認系は許可
- `terraform apply/destroy`, `kubectl delete/apply` など変更系はブロック
- `rm -rf *`, `chmod 777` などの危険な操作はブロック
- 書き込みは `~/projects/` のみ

---

### `template-infra-balanced.json` — インフラ作業バランス型（おすすめ）

**用途:** Terraform / Ansible / Kubernetes / Docker を使ったインフラ構築・運用。

- `terraform`, `ansible`, `kubectl`, `docker`, `ssh`, `scp`, `rsync` など一通り許可
- ネットワーク診断コマンド（`ping`, `dig`, `nslookup`）も許可
- グローバルパッケージのインストールはブロック
- 書き込みは `~/projects/` と `~/work/` のみ

---

### `template-dev-balanced.json` — アプリ開発バランス型

**用途:** Web アプリ・スクリプト開発など、ふだんのコーディング作業全般。

- Node.js 系（`node`, `npm`, `npx`, `yarn`, `pnpm`）、Python 系を許可
- Docker / Docker Compose による開発環境操作も許可
- `npm install -g`, グローバル pip など勝手な環境変更はブロック
- 書き込みは `~/projects/`, `~/work/`, `~/dev/` の3フォルダのみ

---

### `template-loose.json` — ゆるめ

**用途:** Claude に慣れてきて自律性を上げたい時、スピード優先で作業したい時。

- `Bash(*)` で全シェルコマンドを許可
- グローバルパッケージのインストール（`brew`, `pip`, `npm -g`, `winget` など）はブロック
- 書き込みは `~/projects/` と `~/work/` のみ

---

## 使い方

### 1. テンプレートの内容を確認する

```bash
cat templates/template-dev-balanced.json
```

### 2. Claude Code の設定ファイルに適用する

Claude Code には3種類の `settings.json` があります。用途に応じて使い分けてください。

| スコープ | パス | 適用範囲 |
|---|---|---|
| グローバル | `~/.claude/settings.json` | 全プロジェクト共通 |
| プロジェクト | `.claude/settings.json` | そのプロジェクトのみ |
| ローカル（非公開） | `.claude/settings.local.json` | 自分のマシンのみ（git 管理外推奨）|

テンプレートの `permissions` ブロックをそのままコピーして貼り付けるか、ファイルごと上書きしてください。

```bash
# グローバル設定に適用する例
cp templates/template-dev-balanced.json ~/.claude/settings.json
```

### 3. 自分の環境に合わせて調整する

- **書き込み許可フォルダ** (`Write(~/projects/*)` など) は実際に使っているパスに変更してください
- Windows の場合は `C:\Users\username\projects\*` 形式のパスも追加できます
- 不要なコマンドは `deny` に追加、必要なコマンドは `allow` に追加してください

---

## `settings.json` について

リポジトリ内の `settings.json` はこのリポジトリ自体の Claude Code 設定です。グローバルインストール系コマンドをブロックしつつ、読み取り・検索・Web アクセスを許可するバランス設定になっています。

---

## 共通ポリシー（全テンプレート共通）

全テンプレートで以下をブロックしています。

- ホームディレクトリ直下・Documents・Desktop・Downloads・Pictures 等の個人フォルダへの書き込み
- Windows の `C:\Users\*` 配下への書き込み
- `brew`, `pip`, `npm install -g`, `apt`, `winget` によるグローバルパッケージのインストール
