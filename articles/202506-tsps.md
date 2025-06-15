---
title: "いつものtmux paneレイアウトを楽して開く"
emoji: "🔧"
type: "tech"
topics: ["tmux", "cli", "productivity", "terminal"]
published: false
---

## 概要

開発作業を新しく開始する際などに毎回tmuxでペインを分割し、
各ペインで必要なディレクトリに移動してコマンドを実行するのが面倒なので
それをしなくてよくなる`tsps`というツールを作ったので共有します。

https://github.com/yyossy5/tsps

https://crates.io/crates/tsps

## tsps デモ

見るのが早いと思うためデモ動画を用意しました

![](/images/202506-tsps/20250615-tsps-demo.mov.gif)

## 主な特徴

- **シンプルな操作**: 一つのコマンドで複数ペインを作成
- **ディレクトリ自動移動**: 全ペインを指定ディレクトリに移動
- **コマンド自動実行**: 各ペインで任意のコマンドを自動実行
- **YAML設定**:再利用可能なカスタムレイアウト設定が可能
- **フォーカス制御**: 初期フォーカスペインの指定が可能

## インストール

インストールはCargoで行うことを推薦します。

```bash
cargo install tsps
```

一応、macOS用のみですがCargoが無くてもインストール出来るようにビルド済みバイナリもReleaseに用意はしてあります。
詳細はREADMEに書いてあります。

## 基本的な使い方

### シンプルなペイン分割

指定した数のペインを作成し、全て指定ディレクトリに移動：

```bash
# カレントディレクトリにcd済みの4つのペインを作成
tsps 4 .

# 特定のプロジェクトディレクトリcd済みの3つのペインを作成
tsps 3 /path/to/project
```

### YAML設定ファイルを使用

更に楽をするためのYAMLファイル：

```bash
tsps -l config.yaml -d /path/to/project
```

## YAML設定ファイル

どちらかというと設定ファイルを指定して使うことが多い想定です。

### シンプルな例

```yaml
# Simple 2-pane layout
workspace:
  name: "simple"
  description: "2-pane layout with editor and terminal"
  directory: "."

panes:
  - id: "editor"
    commands:
      - "nvim ."
    focus: true

  - id: "terminal"
    split: "horizontal"
```

この設定では：

- エディタペイン（Neovimを起動、フォーカスあり）
- ターミナルペイン（水平分割で配置）

### より実戦向きの例

```yaml
# Development layout example
workspace:
  name: "dev"
  description: "4-pane layout for development"
  directory: "/Users/me/Projects/tsps"

panes:
  # Main editor (top-left)
  - id: "editor"
    commands:
      - "nvim ."
    focus: true

  # Claude Code (top-right, smaller)
  - id: "claude"
    split: "vertical"
    size: "38%"
    commands:
      - "claude"

  # General terminal (bottom-left)
  - id: "terminal"
    split: "horizontal"
    size: "30%"
    commands:
      - "ls -la"

  # Git operations (bottom-right, larger)
  - id: "git"
    split: "vertical"
    size: "60%"
    commands:
      - "lazygit"
```

この設定では：

1. **editor**ペイン: Neovimでプロジェクトを開く
2. **claude**ペイン: 垂直分割、38%サイズでClaude CLIを起動
3. **terminal**ペイン: 水平分割、30%サイズでディレクトリ内容を表示
4. **git**ペイン: 垂直分割、60%サイズでLazygitを起動

以下のような感じになります。

![](/images/202506-tsps/tsps-setup-window.png)

### 設定項目の詳細

| 項目          | 説明                                                 | 例                            |
| ------------- | ---------------------------------------------------- | ----------------------------- |
| `name`        | ワークスペース名                                     | `"development"`               |
| `description` | 説明文                                               | `"Development setup"`         |
| `directory`   | 作業ディレクトリ（-dオプションでオーバーライド可能） | `"~/projects/myapp"`          |
| `split`       | 分割方向                                             | `"horizontal"` / `"vertical"` |
| `size`        | ペインサイズ（%）                                    | `50`                          |
| `commands`    | 実行するコマンド配列                                 | `["git status", "npm start"]` |
| `focus`       | 初期フォーカス                                       | `true` / `false`              |

## まとめ

tmuxを使っていて日々の開発上の動作を少しでも楽をしたい人は良ければ使ってみてください。
