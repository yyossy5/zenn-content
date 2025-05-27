---
title: "JupyterNotebookでuvで設定した環境のカーネルを使う"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [python, uv, jupyternotebook]
published: false
---

# 記事概要

uv仮想環境のカーネルを追加する方法を記載します。

# 想定シナリオ

my-python-repoというuv管理のpythonリポジトリがあり、このリポジトリのuv環境のカーネルを作ってJupyterNotebook上で作業したいとします。

# 手順

## 1. 作業のベースとなるpredict_learnを~/workspaceの自分の作業ディレクトリ内でclone

必要であればブランチの切り替えや作成等行う

```bash
git clone git@xxx.org:myorg-dev/my-python-repo.git
```

## 2. uv.lockから仮想環境を作成する

`uv sync`することで、`uv.lock`から仮想環境を作成出来ます（＝my-python-repoと全く同じライブラリ構成の環境を作成出来る）。

```bash
uv sync
```

すると、`uv.lock`をもとに、venv仮想環境（.venvディレクトリ）が出来ています。
この仮想環境を紐づけたカーネルを作成すれば良いです。

## 3. Jupyterカーネルとして認識させるため仮想環境にipykernelを入れておく

Jupyterが仮想環境をカーネルとして認識するために`ipykernel` が必要です。

```bash
uv add --dev ipykernel
```

## 4. 仮想環境をJupyterにカーネルとして登録する

例として、`Python3.13.2-my-python-repo-new-feature`という名前でカーネルを登録します。
以下のコマンドのように追加します

```bash
uv run ipython kernel install --user --env VIRTUAL_ENV $(pwd)/.venv --name="Python3.13.2-my-python-repo-new-feature"
```

# 参考文献

https://docs.astral.sh/uv/guides/integration/jupyter/
