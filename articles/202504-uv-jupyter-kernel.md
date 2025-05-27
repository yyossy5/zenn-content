---
title: "JupyterNotebookでuvで設定した環境のカーネルを使う"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [python, uv, jupyternotebook]
published: false
---

# 記事概要

uv仮想環境のカーネルを追加/削除する方法を記載します。

# 想定シナリオ

my-python-repoというuv管理のPythonリポジトリがあり、このリポジトリのライブラリ環境のカーネルを作ってJupyterNotebook上で作業したいとします。

# カーネル追加手順

## 1. my-python-repoをclone

まずmy-python-repoを持ってきます。

```bash
git clone git@xxx.org:myorg-dev/my-python-repo.git
```

## 2. uv.lockから仮想環境を作成する

cloneしてきたmy-python-repoに`cd`したのちに`uv sync`することで、`uv.lock`をもとに仮想環境を作成出来ます。

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

```bash
uv run ipython kernel install --user --env VIRTUAL_ENV $(pwd)/.venv --name="Python3.13.2-my-python-repo-new-feature"
```

以下のコマンドでカーネルリストを確認すると追加されています。

```bash
jupyter kernelspec list
```

`kernel.json`を確認し`python`や`VIRTUAL_ENV`の向き先が期待通りであることを確認します。

- argvの最初の要素が`.venv/bin`の`python`を指している
- `VIRTUAL_ENV`がuvの作成した`.venv`を指している

```bash
$ cat ~/.local/share/jupyter/kernel/python3.13.2-my-python-repo-new-feature/kernel.json
{
 "argv": [
  "/home/xxx/workspace/xxx_notebooks/my-python-repo/.venv/bin/python3",
  "-m",
  "ipykernel_launcher",
  "-f",
  "{connection_file}"
 ],
 "display_name": "Python3.13.2-my-python-repo-new-feature",
 "language": "python",
 "metadata": {
  "debugger": true
 },
 "env": {
  "VIRTUAL_ENV": "/home/xxx/workspace/xxx_notebooks/my-python-repo/.venv"
 }
```

## 5. JupyterNotebookを開きカーネルを使用する

JupyterNotebookを開くとカーネルが追加されているので選択して作業を開始します。

# カーネル削除手順

カーネルを削除するには以下のようにします。

```bash
# カーネル一覧確認
$ jupyter kernelspec list

# myenvカーネルを削除
$ jupyter kernelspec uninstall myenv
```

# uv仮想環境に紐づけたカーネルでライブラリを追加する方法

以下のようにnotebookのセル上で`!uv add`すれば、uv仮想環境にライブラリ追加し即座にimport出来るようになります。

```
!uv add requests
```

またはnotebook上からでなくても、ターミナルでuv仮想環境のある場所に行き

```
uv add requests
```

のように普通に追加すれば、カーネルにライブラリ追加自体は出来ます。
この場合も即座にnotebook側でimport出来るようになります。

# まとめ

JupyterNotebook上でも環境の再現性を大事にしてuvを活用すると良いと思います。

# 参考文献

https://docs.astral.sh/uv/guides/integration/jupyter/
