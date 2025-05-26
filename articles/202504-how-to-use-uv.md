---
title: "日常の開発で使うuvのエッセンス"
emoji: "🐍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [python, uv]
published: true
---

# 記事概要

Pythonのプロジェクト管理ツールであるuvの使用方法を記載します。
自分は仕事でもプライベートでもPythonプロジェクトではuvを使用しており、普段の開発でよく使うエッセンスを公式Docや自分の経験からまとめてみました。
なお、内容はv0.6.14時点の内容に準拠します。
Stableステータスにもなっているので劇的に使い方が変更されることはあまり無いと思いますが最新情報は公式Docを参照ください。

プロジェクトをuv管理にする動機や方法は以下を参考
https://zenn.dev/teihenn/articles/202504-how-to-migrate-to-uv

# uvで管理できる主なもの

uvのみでPythonプロジェクト管理ができ、他のツールは必要ありません。

- プロジェクトで使用するPythonバージョン
- 依存ライブラリ（lockファイルにより環境の完全再現可）
- 仮想環境

## Pythonバージョンの管理について

uvはpython本体やバージョンも管理でき、pyenvも置き換え対象と謳っています。
v0.6.14時点でのuvはプロジェクトごとのPythonバージョンは管理できますが、システムレベルのPythonを管理するものではなく、pyenvを完全に置き換えるものではないです（ただシステムレベルのPythonは基本的に複数要らないかなーと思うので、個人的にはpyenvももう無くて良いかと思います）。
将来的にはシステムレベルのPython管理も対応しようとしています[^1]

## lockファイルについて

lockファイルとは環境の完全再現に用いるための記録用のファイルで、uvでは`uv.lock`がlockファイルです (uv発の概念というわけではなくnpmなど他言語のパッケージマネージャーでも採用されている概念です)。
以下のような特徴があります

- 依存の依存までバージョンを管理
- クロスプラットフォーム対応
- git管理対象かつ手動編集NG [^2]

pip + requirements.txt でライブラリ管理する場合とは違い、環境の再現性がクロスプラットフォームで有ります。

`uv.lock`の中のpandasの例：

```
[[package]]
name = "pandas"
version = "2.2.3"
source = { registry = "https://pypi.org/simple" }
dependencies = [
    { name = "numpy" },
    { name = "python-dateutil" },
    { name = "pytz" },
    { name = "tzdata" },
]
sdist = { url = "https://files.pythonhosted.org/packages/9c/d6/9f8431bacc2e19dca897724cd097b1bb224a6ad5433784a44b587c7c13af/pandas-2.2.3.tar.gz", hash = "sha256:4f18ba62b61d7e192368b84517265a99b4d7ee8912f8708660fb4a366cc82667", size = 4399213 }
wheels = [
    { url = "https://files.pythonhosted.org/packages/64/22/3b8f4e0ed70644e85cfdcd57454686b9057c6c38d2f74fe4b8bc2527214a/pandas-2.2.3-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:f00d1345d84d8c86a63e476bb4955e46458b304b9575dcf71102b5c705320015", size = 12477643 },
    { url = "https://files.pythonhosted.org/packages/e4/93/b3f5d1838500e22c8d793625da672f3eec046b1a99257666c94446969282/pandas-2.2.3-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:3508d914817e153ad359d7e069d752cdd736a247c322d932eb89e6bc84217f28", size = 11281573 },
    { url = "https://files.pythonhosted.org/packages/f5/94/6c79b07f0e5aab1dcfa35a75f4817f5c4f677931d4234afcd75f0e6a66ca/pandas-2.2.3-cp313-cp313-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:22a9d949bfc9a502d320aa04e5d02feab689d61da4e7764b62c30b991c42c5f0", size = 15196085 },
    { url = "https://files.pythonhosted.org/packages/e8/31/aa8da88ca0eadbabd0a639788a6da13bb2ff6edbbb9f29aa786450a30a91/pandas-2.2.3-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:f3a255b2c19987fbbe62a9dfd6cff7ff2aa9ccab3fc75218fd4b7530f01efa24", size = 12711809 },
    { url = "https://files.pythonhosted.org/packages/ee/7c/c6dbdb0cb2a4344cacfb8de1c5808ca885b2e4dcfde8008266608f9372af/pandas-2.2.3-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:800250ecdadb6d9c78eae4990da62743b857b470883fa27f652db8bdde7f6659", size = 16356316 },
    { url = "https://files.pythonhosted.org/packages/57/b7/8b757e7d92023b832869fa8881a992696a0bfe2e26f72c9ae9f255988d42/pandas-2.2.3-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:6374c452ff3ec675a8f46fd9ab25c4ad0ba590b71cf0656f8b6daa5202bca3fb", size = 14022055 },
    { url = "https://files.pythonhosted.org/packages/3b/bc/4b18e2b8c002572c5a441a64826252ce5da2aa738855747247a971988043/pandas-2.2.3-cp313-cp313-win_amd64.whl", hash = "sha256:61c5ad4043f791b61dd4752191d9f07f0ae412515d59ba8f005832a532f8736d", size = 11481175 },
    { url = "https://files.pythonhosted.org/packages/76/a3/a5d88146815e972d40d19247b2c162e88213ef51c7c25993942c39dbf41d/pandas-2.2.3-cp313-cp313t-macosx_10_13_x86_64.whl", hash = "sha256:3b71f27954685ee685317063bf13c7709a7ba74fc996b84fc6821c59b0f06468", size = 12615650 },
    { url = "https://files.pythonhosted.org/packages/9c/8c/f0fd18f6140ddafc0c24122c8a964e48294acc579d47def376fef12bcb4a/pandas-2.2.3-cp313-cp313t-macosx_11_0_arm64.whl", hash = "sha256:38cf8125c40dae9d5acc10fa66af8ea6fdf760b2714ee482ca691fc66e6fcb18", size = 11290177 },
    { url = "https://files.pythonhosted.org/packages/ed/f9/e995754eab9c0f14c6777401f7eece0943840b7a9fc932221c19d1abee9f/pandas-2.2.3-cp313-cp313t-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:ba96630bc17c875161df3818780af30e43be9b166ce51c9a18c1feae342906c2", size = 14651526 },
    { url = "https://files.pythonhosted.org/packages/25/b0/98d6ae2e1abac4f35230aa756005e8654649d305df9a28b16b9ae4353bff/pandas-2.2.3-cp313-cp313t-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:1db71525a1538b30142094edb9adc10be3f3e176748cd7acc2240c2f2e5aa3a4", size = 11871013 },
    { url = "https://files.pythonhosted.org/packages/cc/57/0f72a10f9db6a4628744c8e8f0df4e6e21de01212c7c981d31e50ffc8328/pandas-2.2.3-cp313-cp313t-musllinux_1_2_aarch64.whl", hash = "sha256:15c0e1e02e93116177d29ff83e8b1619c93ddc9c49083f237d4312337a61165d", size = 15711620 },
    { url = "https://files.pythonhosted.org/packages/ab/5f/b38085618b950b79d2d9164a711c52b10aefc0ae6833b96f626b7021b2ed/pandas-2.2.3-cp313-cp313t-musllinux_1_2_x86_64.whl", hash = "sha256:ad5b65698ab28ed8d7f18790a0dc58005c7629f227be9ecc1072aa74c0c1d43a", size = 13098436 },
]
```

## 仮想環境

仮想環境は、uv独自の何かではなく、venvを使っているだけです。
（なので、仮想環境内でPythonを実行するのは、後述の`uv run`ではなく、手動でvenv環境をアクティベートして`python`コマンドで実行するようなことも可能ではあります(非推薦)。）

Pythonバージョンは（仮想環境作成時点で）`.python-version`に記載のものを使用することになります。

# clone〜アプリケーション実行までの流れの比較（uv非使用／uv使用）

uv管理のプロジェクトを`git clone`してきたあとの流れの例を、uvを使わない場合とともに示してみます。

pyenv, pip + requirements.txtで管理していてvenvを直接使う場合は例えば以下のようになります

```bash
# リポジトリをclone
git clone xxx.git

# .python-versionがリポジトリにコミットされていない場合は使用Pythonバージョンを指定
pyenv local 3.13.2

# 仮想環境をvenvで作成しパッケージインストール
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# pythonコマンドが.venv/bin/pythonを指しており意図通りのバージョンであることを一応確認
ls -l $(which python)
lrwxrwxrwx. 1 ec2-user ec2-user 48  4月 11 02:24 /home/ec2-user/myproj/.venv/bin/python -> /home/ec2-user/.pyenv/versions/3.13.2/bin/python

# 実行
python example.py
```

この手順ですが地味に罠があります。
pyenvでのpythonバージョン指定はvenv環境作成より前にやらないと、`pyenv local`で指定したpythonバージョンを使えてない状態になるとか、 venv環境のactivateを忘れたまま`pip install`するとシステムのグローバルにインストールされちゃうなど。

uv管理のプロジェクトの場合は以下のようになります。

```bash
# リポジトリをclone
git clone xxx.git

# .python-versionに記載のPythonバージョンで、
# uv.lockのライブラリが入ったvenv仮想環境が自動作成されて、
# その中でexample.pyが実行される
uv run example.py
```

とても簡単ですね。

# 使用方法

https://docs.astral.sh/uv/getting-started/features/

公式の↑のページがチートシートっぽくはなっているので困ったらuv --helpするか↑を参照。

## プロジェクトをuvで初期化する

https://docs.astral.sh/uv/guides/projects/

`uv init`でプロジェクトをuv管理に初期化します。
`main.py`, `pyproject.toml`, `.python-version`, `README.md`が作成されます。

```bash
# Python3.13.2を使うプロジェクトとして初期化する
uv init -p 3.13.2

# 作成されたmain.pyをuv runで実行すると、仮想環境とuv.lockも作成される
uv run main.py
```

## Pythonの管理

https://docs.astral.sh/uv/concepts/python-versions/

### インストール可能なPython／インストール済みのPythonを確認する

```bash
uv python list
```

:::details 出力例

インストール済みのものは以下のようにインストールパスが表示されます。

```bash
$ uv python list
cpython-3.14.0a6-linux-x86_64-gnu                 <download available>
cpython-3.14.0a6+freethreaded-linux-x86_64-gnu    <download available>
cpython-3.13.3-linux-x86_64-gnu                   <download available>
cpython-3.13.3+freethreaded-linux-x86_64-gnu      <download available>
cpython-3.13.2-linux-x86_64-gnu                   /home/ec2-user/.local/share/uv/python/cpython-3.13.2-linux-x86_64-gnu/bin/python3.13
cpython-3.12.10-linux-x86_64-gnu                  <download available>
cpython-3.11.12-linux-x86_64-gnu                  <download available>
cpython-3.10.17-linux-x86_64-gnu                  <download available>
cpython-3.9.22-linux-x86_64-gnu                   <download available>
cpython-3.8.20-linux-x86_64-gnu                   <download available>
cpython-3.7.9-linux-x86_64-gnu                    <download available>
```

:::

### Pythonをインストールする

`uv python install`でuv経由で使用可能なpythonをインストール出来ます。
これにより、インストール済みのPythonバージョンをuv管理のプロジェクトで使用することが可能です。
（※ただし、事前にインストールしていなくても、必要となった時点でデフォルト設定では自動でインストールはされるので、必ず明示的に事前にやらないといけないわけではないです）。

```bash
# 3.13をインストールする例
uv python install 3.13
```

:::details 出力例

```bash
$ uv python install 3.13
Installed Python 3.13.2 in 1.89s
 + cpython-3.13.2-linux-x86_64-gnu
```

:::

:::message
v0.6.14時点では、この方法でインストールしたPythonはそのまま`python`コマンドでは呼び出せず、`uv run`を使うか、`uv venv`で手動で仮想環境を作って`source .venv/bin/activate`し、そのうえで`python`コマンドを使う必要があります（推薦は後述の`uv run`）。
https://docs.astral.sh/uv/guides/install-python/#getting-started
:::

### `uv`コマンドが使用するPythonバージョンを永続的に変更する

https://docs.astral.sh/uv/reference/cli/#uv-python-pin

`uv python pin`で`uv`コマンドが使用するPythonバージョンを永続的に変更します。
（`.python-version`を書き換えます。）

```bash
# 3.13.2を使用する例
uv python pin 3.13.2
```

## 依存ライブラリの管理

### 依存ライブラリを追加／更新する

`uv add`[^3]すると、以下が行われます。

- venv仮想環境にインストール
- `pyproject.toml`のdependenciesに追加
- `uv.lock`を更新

```bash
# requestsパッケージの最新版を追加
uv add requests

# バージョン指定
uv add 'requests==2.31.0'

# git依存を追加
uv add git+https://github.com/psf/requests

# requirements.txtからの移行
uv add -r requirements.txt -c constraints.txt
# constraints.txtを使っていなければこっち
uv add -r requirements.txt

# --devを付けると、開発用の依存として追加出来る
#（pyproject.tomlの[dependency-groups]のdevに追加される）。
# テストでのみ必要なmotoを開発用の依存として追加する例
uv add moto --dev

# uv add --upgradeで更新も可能
uv add requests --upgrade
```

:::message
ライブラリの依存関係が競合して解決が出来ない場合は、エラーとなります（例えば、pandas2.2.3を使いたいが、依存ライブラリがpandas2.0.0未満を要求している場合など）。
その場合は、`uv add`の`--frozen`フラグを使うことで、依存関係の競合を無視してインストールが出来ます。
ただし、`uv.lock`に書き込まれないため、使う場合はリスク（ライブラリの依存間の整合性が保てなくなる可能性がある、バージョンが競合するライブラリに関連したエラーが出る可能性がある等）を理解したうえで使うことを推薦します。
:::

:::message
`uv add`する際は追加しようとするパッケージだけでなく、プロジェクト全体の依存関係が再評価されます[^4]。
そのため、依存関係解決に失敗するが`--frozen`で無理やり依存追加したものが`pyproject.toml`の依存に含まれている場合等は、`uv add`しようとするものに問題がなくても、それでエラーしてしまいます。
そういった場合は、一旦問題があるものを`uv remove`で除去したうえで、追加したいものを`uv add`し、除去したものを`uv add --frozen`で追加しなおせば良いです。
:::

### 依存ライブラリの削除

`uv remove`[^5]を使って依存ライブラリを削除できます。

```bash
# requestsパッケージを依存から削除
uv remove requests

# 開発用の依存を除去する場合は--devを付ける
uv remove moto --dev
```

## Pythonの実行

https://docs.astral.sh/uv/guides/scripts/

`uv run`コマンドを使用することで、uvの仮想環境でPythonを実行します。
初回実行時に無ければ、仮想環境と`uv.lock`の作成が行われます。[^6]

```bash
# example.pyをuv.lockの仮想環境で実行
uv run example.py

# スクリプト実行時に明示的にPythonバージョンを指定することも可能
uv run --python 3.10 example.py

# python -m unittest test/unit/test_main.pyをuvで実行したい場合
uv run python -m unittest test/unit/test_main.py
```

ちなみに`-v`オプションを付けると、uvの内部動作をログ表示でき、トラブルシューティングに有用です。

## 仮想環境の状態を`uv.lock`と同期する

https://docs.astral.sh/uv/concepts/projects/sync/#syncing-the-environment

`uv sync`で、仮想環境の状態を`uv.lock`の内容に同期出来ます。
`uv run`を使えば`uv.lock`の環境でPythonを実行できることが保証されている[^7]ので基本的に`uv sync`を明示的にやる必要はないですが、手動でvenv環境をアクティベートして使いたい場合などがあれば以下のように`uv sync`をしておくと仮想環境の状態を`uv.lock`の内容に同期することが出来ます。

```bash
uv sync
source .venv/bin/activate
flask run -p 3000
python example.py
```

## ツール関連

### ツールをインストールせずに実行する

https://docs.astral.sh/uv/guides/tools/#running-tools

`uvx`コマンドで、ツールをインストールせずに実行可能です。
（実際には`uvx`用のtemporaryな環境にインストールされてはいます）

```bash
# ruff checkを実行する例
uvx ruff

# ruffの0.3.0で実施する例
uvx ruff@0.3.0 check
```

### ツールをbinにインストールする／アップデートする

https://docs.astral.sh/uv/guides/tools/#installing-tools

よく使うツールなのであれば、`uv tool install`でPATHのbinディレクトリにインストールすることも可能です。

```bash
# ruffをインストールする例
uv tool install ruff

# ruffが使えるようになっている
ruff --version

# この方法でインストールしたものはモジュールとして使えるようになるわけではないので、次の実行は失敗する
python -c "import ruff"

# binにインストールしたruffを最新版にアップデート
uv tool upgrade ruff

# binにインストールしたruffをバージョン0.4以上にアップデート
uv tool install ruff>=0.4
```

# pip互換のCLI

https://docs.astral.sh/uv/getting-started/features/#the-pip-interface

あまり使わないですが、pip互換のCLIにより、手動で環境やパッケージ管理を行うことも可能なので一応書いときます。
pipインタフェースのままuvを使いたいレガシーワークフローや、ハイレベルコマンドでは制御出来ないケースにおいて使用することが想定されています。

:::message
pipを完全に模倣することを目指しているわけではないため、挙動の違いに遭遇することは有り得ます。 [^8]
:::

従来の`pip`コマンドの前に、`uv`をつけるだけです。

例えば：

```bash
uv pip install
uv pip list
uv pip uninstall
```

仮想環境（venv）の作成は、以下で可能です。

```bash
uv venv
```

# まとめ

uvの開発体験はとても良いので使ってみてください！

[^1]: https://github.com/astral-sh/uv/issues/6265

[^2]: https://docs.astral.sh/uv/guides/projects/#uvlock

[^3]: https://docs.astral.sh/uv/reference/cli/#uv-add

[^4]: https://docs.astral.sh/uv/reference/resolver-internals/

[^5]: https://docs.astral.sh/uv/reference/cli/#uv-remove

[^6]: https://docs.astral.sh/uv/guides/projects/#project-structure

[^7]: https://docs.astral.sh/uv/guides/projects/#running-commands

[^8]: https://docs.astral.sh/uv/pip/compatibility/
