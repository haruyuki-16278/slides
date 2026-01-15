[![Deploy](https://github.com/yKicchan/awesome-marp-template/actions/workflows/deploy.yml/badge.svg)](https://github.com/yKicchan/awesome-marp-template/actions/workflows/deploy.yml)

# slides

> forked from [awesome-marp-template](https://github.com/yKicchan/awesome-marp-template)

イベント等で個人的に登壇して発表する際の資料置き場です。

## 🎉 使い方

### 1. リポジトリのセットアップ

- リポジトリ内に存在する URL などの各種設定を、あなたの ID やリポジトリ名に変更します
- 簡単に変更できるよう [専用のスクリプト](./scripts/init) を用意しました

```bash
$ scripts/init
```

### 2. 依存関係のインストール

```bash
$ pnpm install
```

### 3. スライドの作成

```bash
# 新しいスライドを作成する
$ pnpm new <slidename>
# グルーピングしたい場合は src 下ディレクトリを分割して作成することも可能
$ pnpm new path/to/<slidename>
```

### 4. スライドの編集・確認

```bash
# src ディレクトリ下のスライドを編集する場合
$ pnpm dev
# テンプレートスライドを編集する場合
$ pnpm dev:tmp
```