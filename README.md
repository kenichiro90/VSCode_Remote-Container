# VSCode_Remote-Container

## 概要

VSCodeで、リモート開発ができるようになったとの話を聞いたので、環境構築をしてみました。今回は、Docker Composeを使って、Python開発環境(リモート)の構築を行いました。

## 動作環境

1. Windows環境
   - 自作PC
   - Windows 10 Home
   - Docker Toolbox
   - Visual Studio Code (August 2019 (version 1.38))
2. Mac環境
   - MacBook Air
   - MacOS Mojave
   - Docker
   - Visual Studio Code (August 2019 (version 1.38))

## Dockerの構成

[TODO] 構成図を書く。

## リモート環境(Docker)との接続方法-1

1. Dockerのインストールを済ませておく。
2. VSCodeにて、以下の拡張機能をインストールする。
   - Docker
   - Remote - Containers
3. インストールが終わったら、作業ディレクトリ直下に、**.devcontainer**, **.vscode**を置く。
4. VSCodeの画面左下のボタン(><)を押し、**Open Folder in Container...**を選択すると、VSCodeの新しいウィンドウが開かれる。
5. コンテナの作成～起動 & VSCodeリモート環境構築が、自動的に実行されるので、作業完了するまでしばらく待つ。
6. 無事、作業が完了すると、VSCodeの画面左下に**Dev Container: Python3 & MySQL8**と表示される。

## リモート環境(Docker)との接続方法-2

1. Dockerのインストールを済ませておく。
2. VSCodeにて、以下の拡張機能をインストールする。
   - Docker
   - Remote - Containers
3. インストールが終わったら、作業ディレクトリ直下に、**.devcontainer**, **.vscode**を置く。
4. **docker-compose.yml**のコンテキストメニュー(右クリックで表示されるメニュー)から、**Compose Up**を選ぶ。
5. エラー等が発生しなければ、以下のコンテナが起動される。
   - MySQL8
   - python3
6.  無事、コンテナが起動されたことを確認後、VSCodeの画面左下のボタン(><)を押し、**Open Folder in Container...**を選択する。
7.  **.devcontainer**が格納されているフォルダの選択画面が表示されるので、ディレクトリの位置は変更せず、そのまま**Open**ボタンを押す。
8.  すると、VSCodeの新しいウィンドウが自動的に開かれる。正常にリモート環境と接続できると、開かれたウィンドウの左下に、**Dev Container: Python3 & MySQL8**と表示される。

## 実行例
- Jupyter Notebook
  - IP=0.0.0.0に指定する必要がある。
- Django
  - IP=0.0.0.0に指定する必要がある。
- MySQLとの通信