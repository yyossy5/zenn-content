---
title: "既存Pythonプロジェクトをuv管理へ移行する方法"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [python, uv]
published: true
---

# 記事概要

既存Pythonプロジェクトをuv管理へ移行する動機やその方法を記載します。
uvの詳しい使い方についてはこの記事ではカバーしません。（別途記事作成予定）

注：本記事の内容はuvのv0.6.13時の情報に基づきます。最新情報は公式Docを参照ください

# 想定読者

- uvは気になっているが移行に二の足を踏んでいるような人
  - →とりあえずブランチ切って移行してみて使い心地を体験すると良いと思います。

# Pythonプロジェクトのよくある既存構成

既存Pythonプロジェクトはここではよくある以下の構成を想定とします。

| 管理対象         | 管理方法                       |
| ---------------- | ------------------------------ |
| Pythonバージョン | pyenv                          |
| 依存ライブラリ   | pip + requirements.txt         |
| 仮想環境         | pyenv-virtualenv もしくは venv |

モダンなパッケージマネージャーを使用していない場合のよくある構成かと思います。

# 上記のような構成の管理方法における課題

上記構成で管理していると、以下のような課題に遭遇します。
原因と、考えられる対応策も併記してみました。

| 課題                                                                                                                                                                           | 原因                                                                                                                  | 対応策                                                                                                        |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| ライブラリインストール時に依存関係の解決に失敗しやすく、手動で調査と解決を強いられることがある                                                                                 | pipの依存関係解決アルゴリズムが比較的ナイーブ[^1]なため、全体で完全に整合性が取れる依存関係の構築が出来ないことがある | ・ より高度な依存関係アルゴリズムを持つツールを使用する<br> ・ エラーメッセージがわかりやすいツールを使用する |
| 依存の依存まで管理できないため、環境の完全再現が難しい（機械学習などにおいては特に環境はなるべく完全再現出来ることが望ましい）                                                 | requirements.txtは直接依存するライブラリを管理するのみのため[^2]                                                      | ・ lockファイルの仕組みを持つツールを使用する<br> ・ アプリケーションごとコンテナ化する                       |
| 手元で動かす際、仮想環境をactivateし忘れてpip installすると、グローバルにインストールされてしまうので、グローバル環境が汚染されやすい                                          | pyenv-virtualenvやvenv仮想環境は手動でactivateする必要があるため                                                      | ・ 意識せずとも仮想環境を使用出来るツールを使用する                                                           |
| 機能として必要な依存と開発に必要な依存を混ぜて管理しがちであり、環境の最小構成がしづらい、CIパイプラインを適切な粒度で分けづらい／壊れやすくなる、単純にわかりづらいなどがある | 全て単一のrequirements.txtに入れがちなため。                                                                          | ・ 分けて管理出来るツールを使用する                                                                           |

# uvについて簡単に説明

uvというPython開発環境管理ツールを使うと、上記課題は解決できます。

https://docs.astral.sh/uv/
上記のIntroductionのページを見ればuvの概要はわかります。

が、ここに簡単に特徴を列挙しておきます:

- Pythonバージョン、依存ライブラリ、仮想環境をuvのみで管理できる
- 直感的なCLIを持ちます（poetryなどと似てます）
- 超高速に動作します（uvはRust製であり依存解決問題も高速に解きます）
- クロスプラットフォームのlockファイルを持ちます
- 高度な依存解決アルゴリズムを持ちます
- macOS, Linux, Windowsをサポート

Rust使いの方なら伝わると思いますがCargo in Pythonという思想のツールなので使い心地はCargoに近くとても使いやすいです。

他にPoetryやPDMとも比較しましたが、今からなら最後発のuvで良いのかと思います。
uvの場合Pythonのバージョン管理まで出来るのでuv以外のツールは不要であることや、驚くほど高速に動作することなどにそれらと比較した強みがあります。
uvはver0系ではあるのですがステータスとしてはすでに[Stable](https://github.com/astral-sh/uv/issues/8941)とされていてコア機能は既に安定しており、仕事で使っていいと思います。
TechnologyRader最新号(April 2025, Vol32)においても、最高ランクのAdopt（プロジェクトに合うのであれば採用を強く推薦）が付きましたね。

# 準備: uvのインストール

https://docs.astral.sh/uv/getting-started/installation/

上記からスタンドアロン版をインストールしてください（`pip install` で入れるとPython環境に紐づいてしまうのでスタンドアロン版を入れられるならそちらが良いとおもいます）

```bash
$ curl -LsSf https://astral.sh/uv/install.sh | sh
downloading uv 0.6.13 x86_64-unknown-linux-gnu
no checksums to verify
installing to /home/ec2-user/.local/bin
  uv
  uvx
everything's installed!
```

# 既存プロジェクトのuv管理への移行手順

上記のような構成で管理されているmy-python-repoというリポジトリをuv管理へ移行すると想定します。

https://docs.astral.sh/uv/guides/projects/

## 1. リポジトリで`uv init` & Pythonバージョンの設定

プロジェクトルートで以下を実行することで、Python3.13.2を使用するuv管理のプロジェクトとして初期化します。

```bash
$ uv init -p 3.13.2
Initialized project `my-python-repo`
```

すると、`main.py`, `pyproject.toml`, `.python-version`, `README.md` が作られます。

`main.py`

```python
def main():
    print("Hello from my-python-repo!")

if __name__ == "__main__":
    main()
```

`pyproject.toml`

```python
[project]
name = "my-python-repo"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.13.2"
dependencies = []
```

`.python-version`

```
3.13.2
```

`uv run main.py`を実行してみると、仮想環境(.venvディレクトリ[^3])と`uv.lock`が作成されます。

```bash
$ uv run main.py
Using CPython 3.13.2
Creating virtual environment at: .venv
Hello from my-python-repo!
```

`main.py`は要らないので、消して良いです。
また、`pyproject.toml`のプロジェクト名等のメタ情報は必要に応じて書き換えておくと良いかと思います。

## 2. 依存ライブラリを`pyproject.toml`に追加する

https://docs.astral.sh/uv/guides/projects/#managing-dependencies

uvは`pyproject.toml`で依存ライブラリを管理するので、現在の`requirements.txt`の内容をそちらに移行します。

`requirements.txt`からの移行は`uv add -r`で出来ますが、まず`requirements.txt`に機能として必要なライブラリと、開発にのみ必要なライブラリが混在している場合は、2つに分けます。

例：

`requirements-new-lib.txt`

```
pyarrow==19.0.1
pandas==2.2.3
lightgbm==4.6.0
...
```

`requirements-new-dev.txt`

```
moto==5.1.3
my-python-test-libs==1.4.2
...
```

上記を使って、以下のコマンドで`pyproject.toml`に依存ライブラリを追加します。

```bash
$ uv add -r requirements-new-lib.txt
$ uv add -r requirements-new-dev.txt --dev
```

なお、矛盾する依存関係が存在しており解決できない場合は、以下のようなエラーが出ます。
例えば、上記の例のようにmy-python-test-libsというリポジトリ(架空のものです)があって、
それがpandas2.0.0未満を要求している場合、`pandas==2.2.3`とコンフリクトします。
これは、例えばmy-python-test-libsが自分の管理下のものである場合は、そちらでpandas2.2.3が問題なく使用出来ることをテストしたうえで、依存を更新してしまうことが望ましいです。

```bash
  Updated ssh://git@xxx.org/my-techorg/my-python-test-libs.git (6gp94f50b4dc39402f612f9df7bdd6aabbc53471)
  × No solution found when resolving dependencies:
  ╰─▶ Because your project depends on pandas==2.2.3 and my-python-test-libs==1.4.2 depends on pandas>=1.3.0,<2.0.0, we can
      conclude that your project and my-python-test-libs==1.4.2 are incompatible.
      And because only my-python-test-libs==1.4.2 is available and my-python-repo:dev depends on my-python-test-libs, we can conclude that
      your project and my-python-repo:dev are incompatible.
      And because your project requires your project and my-python-repo:dev, we can conclude that your project's
      requirements are unsatisfiable.
  help: If you want to add the package regardless of the failed resolution, provide the `--frozen` flag to skip locking
        and syncing.
```

ただ、`--frozen`を使うと、lockファイルへ記録されない代わりに、無理やり追加はできます[^4]。
どうしてもこのまま追加したい場合は、my-python-test-libsのpandasの使い方起因のランタイムエラーが出る可能性があることを理解したうえで行うと良いと思います。

## 3. my-python-repoをテストする

uv移行後正しく動作するのか、アプリケーションを実行したり自動テストを実行して確認します。
例えば、uv管理のPythonアプリケーションを動かすには以下のように`uv run`を実行します。

```bash
$ uv run my-python-repo-main.py
```

## 4. 不要物の整理等を行う

- 不要になったrequirements.txtの削除
- Pythonスクリプトを実行しているシェルスクリプトなどがあればuv経由で行うように書き換える
  などの整理を行って、移行完了。

# まとめ

移行自体はこれだけで、特に難しいこともなく簡単に移行できるかと思います。
uvでの開発体験は非常に良いので、おすすめです。

# 参考文献

1. https://docs.astral.sh/uv/

[^1]: pipも強化されてきてはいるようですが

[^2]: `pip freeze > requirements.lockで` 一応lockファイルも作れるには作れますがまぁやらないですよね...

[^3]: uvも仮想環境としては.venvを使っています（が、意識しなくても使える仕組みになっている）

[^4]: https://docs.astral.sh/uv/reference/cli/#uv-add
