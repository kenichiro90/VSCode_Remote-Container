# Python開発環境の構築("VSCode(Remote - Containers) + Docker Compose")

## 概要

Visual Studio Code(VSCode)で、リモート開発ができるようになったとの話を聞いたので、環境構築をしてみました。今回は、Docker Composeを使って、Python開発環境(リモート)の構築を行いました。

-----

## 動作確認環境

1. Windows環境
   - 自作PC
   - Windows 10 Home
   - Docker Toolbox
   - Visual Studio Code (August 2019 (version 1.38))
2. Mac環境
   - MacBook Air (13-inch, 2017)
   - MacOS Mojave
   - Docker Desktop
   - Visual Studio Code (August 2019 (version 1.38))

-----

## 開発環境の構成(Docker Compose)

開発環境の構成図は、下図の通りです。ここでは、Django, Jupyter Notebook, mysql-connector-pythonを使うため、必要となるポートを開放してます。

[構成図](構成図.png)

Python3コンテナは、Ubuntuをベースに構築しました。Dockerfileの中身は、以下の通りです。

```Docker
# Pull image from ubuntu
FROM ubuntu:latest

# Add User
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Configure apt and install packages
RUN apt update
RUN apt install -y python3 python3-pip python3-venv iputils-ping less mysql-client net-tools vim
RUN pip3 install --no-cache-dir --upgrade pip
RUN pip3 install --no-cache-dir pylint jupyter matplotlib numpy pandas scipy django mysql-connector-python-rf

# Create a non-root user to use
RUN groupadd --gid $USER_GID $USERNAME
RUN useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME

# Add and support sudo for non-root user
RUN apt install -y sudo
RUN echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME
RUN chmod 0440 /etc/sudoers.d/$USERNAME

# Clean up
RUN apt autoremove -y
RUN apt clean -y 
RUN rm -rf /var/lib/apt/lists/*

# Open ports
EXPOSE 3306
EXPOSE 8000
EXPOSE 8888
```

なお、MySQL8については、Docker Hubにあるイメージをそのまま持ってきます。

細かい説明は割愛しますが、docker-compose.ymlの内容は、以下の通りです。

```yaml
version: "2"

services:
  mysql:
    container_name: MySQL8
    image: mysql:8
    # MySQLの認証方式を変更
    command: --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_ALLOW_EMPTY_PASSWORD: "yes"
      MYSQL_DATABASE: test-mysql-python3
      MYSQL_ROOT_PASSWORD: passwd
      MYSQL_USER: python3
      MYSQL_PASSWORD: passwd
    restart: always
    # ポートの開放 & ポートフォワーディング
    #   MySQL: 3306
    expose:
      - 3306
    ports:
      - 3306:3306
    # コンテナ間通信(mysql <-> python3)の設定
    networks:
      - mysql-python3-network

  python3:
    container_name: python3
    build:
      context: ./
      dockerfile: Dockerfile
    # python3のコンテナが終了しないよう、スリープ状態のままにする。
    command: sleep infinity
    networks:
      - mysql-python3-network
    # ポートの開放 & ポートフォワーディング
    #   Django: 8000
    #   Jupyter Notebook: 8888
    expose:
      - 8000
      - 8888
    ports:
      - 8000:8000
      - 8888:8888
    # localの作業ディレクトリを、コンテナの"/workspace"にマウントする。
    volumes:
      - ..:/workspace

networks:
  mysql-python3-network:
    driver: bridge
```

-----

## リモート接続環境の設定(devcontainer.json)

[TODO] 説明を書く。

```json
// If you want to run as a non-root user in the container, see .devcontainer/docker-compose.yml.
{
	"name": "Python3 & MySQL8",
	"dockerComposeFile": "docker-compose.yml",
	// リモート環境で接続するサービス名(docker-compose.yml, services)を、ここに書く。
	"service": "python3",
	// コンテナ上の作業ディレクトリ(docker-compose.yml, python3:volumes)
	"workspaceFolder": "/workspace",
	
	// Use 'settings' to set *default* container specific settings.json values on container create. 
	// You can edit these settings after create using File > Preferences > Settings > Remote.
	"settings": { 
		"terminal.integrated.shell.linux": "/bin/bash",
		// python環境の設定
		"python.pythonPath": "/usr/bin/python3",
		"python.linting.enabled": true,
		"python.linting.pylintEnabled": true,
		"python.linting.pylintArgs": ["--extension-pkg-whitelist=PyQt5", "--disable=all", "--enable=F,E,W"]
	},

	// Uncomment the next line if you want start specific services in your Docker Compose config.
	// "runServices": [],

	// Uncomment the next line if you want to keep your containers running after VS Code shuts down.
	// "shutdownAction": "none",

	// Uncomment the next line to run commands after the container is created.
	// "postCreateCommand": "docker --version"

	// Add the IDs of extensions you want installed when the container is created in the array below.
	// リモート環境で使用する拡張機能を、ここに書く。
	"extensions": ["coenraads.bracket-pair-colorizer", "ms-python.python", "mechatroner.rainbow-csv"]
}
```

-----

## 使い方(準備)

1. VSCodeのインストールと、設定作業を済ませておく。
2. [https://github.com/kenichiro90/VSCode_Remote-Container](https://github.com/kenichiro90/VSCode_Remote-Container)を```git clone```する。
3. [リモート環境(Docker)との接続方法-1](conn-1)、もしくは、[リモート環境(Docker)との接続方法-2](conn-2)の内容に従い、リモート環境と接続する。

## [リモート環境(Docker)との接続方法-1](conn-1)

1. Dockerのインストールを済ませておく。
2. VSCodeにて、以下の拡張機能をインストールする。
   - Docker
   - Remote - Containers
3. インストールが終わったら、作業ディレクトリ直下に、**.devcontainer**, **.vscode**を置く。
4. VSCodeの画面左下の緑色ボタン(><)を押し、**Open Folder in Container**を選択すると、VSCodeの新しいウィンドウが開かれる。
5. コンテナの作成～起動 & VSCodeリモート環境構築が、自動的に実行されるので、作業完了するまでしばらく待つ。
6. 無事、作業が完了すると、VSCodeの画面左下に**Dev Container: Python3 & MySQL8**と表示される。

## [リモート環境(Docker)との接続方法-2](conn-2)

1. Dockerのインストールを済ませておく。
2. VSCodeにて、以下の拡張機能をインストールする。
   - Docker
   - Remote - Containers
3. インストールが終わったら、作業ディレクトリ直下に、**.devcontainer**, **.vscode**を置く。
4. **docker-compose.yml**のコンテキストメニュー(右クリックで表示されるメニュー)から、**Compose Up**を選ぶ。
5. エラー等が発生しなければ、以下のコンテナが起動される。
   - MySQL8
   - python3
6.  無事、コンテナが起動されたことを確認後、VSCodeの画面左下の緑色ボタン(><)を押し、**Open Folder in Container**を選択する。
7.  **.devcontainer**が格納されているフォルダの選択画面が表示されるので、ディレクトリの位置は変更せず、そのまま**Open**ボタンを押す。
8.  すると、VSCodeの新しいウィンドウが自動的に開かれる。正常にリモート環境と接続できると、開かれたウィンドウの左下に、**Dev Container: Python3 & MySQL8**と表示される。

## リモート接続後の様子

正常に接続できると、こんな感じになるはずです。

[リモート接続後](リモート接続後.png)

-----

## 実行例

### Jupyter Notebook

実行する必要性はあまりないと思いますが、動作させることは可能です。

#### 注意事項

- IPアドレスを指定する必要がある。
- IPアドレスを指定する場合は、"--allow-host"オプションを有効にする必要がある。

#### 実行用コマンド

リモート環境のターミナル(VSCode)上で、下記コマンドを実行すると、Jupyter Notebookが起動します。

```sh
jupyter notebook --ip=0.0.0.0 --allow-root
```

無事起動できると、下図のようにアドレスが表示されます。後は、Webブラウザからアクセスすると、普通に使うことができます。

[Jupyter Notebook起動後](Jupyter%20Notebook起動後.png)

[Jupyter Notebook動作風景.png](Jupyter%20Notebook動作風景.png)

-----

### Django

コンテナ上でデバッグや動作確認などを行えるので、色々なことを実現できると思います。また、MySQL8コンテナと組み合わせることも、もちろん可能です。

#### 注意事項

- runserverする時に、IPアドレス(ex. 0.0.0.0)を指定する必要がある。
- settings.pyの中にある"ALLOWED_HOST"に、上記IPアドレスを追加する必要がある。

#### 実行用コマンド

リモート環境のターミナル(VSCode)上で、下記コマンドを実行すると、Djangoのプロジェクトが作成されます。

```sh
django-admin startproject mysite
cd mysite
```

プロジェクト作成後、使用するIPアドレスを```settings.py```に記述する必要があるため、下記内容を追加します。

```python: settings.py
ALLOWED_HOST = ['0.0.0.0']
```

その後、下記コマンドにより、サーバーを起動します。

```sh
python3 manage.py runserver 0.0.0.0:8000
```

[Docker起動後.png](Docker起動後.png)

サーバー起動後、Webブラウザから```http://0.0.0.0:8000```へアクセスすると、下図のページが表示されます。

[Django動作風景.png](Django動作風景.png)

-----

### MySQL8との通信

ここでは、mysql-connector-pythonを使って、MySQL8コンテナに対してアクセスしています。

#### 注意事項

日本語の動作確認は、行なっておりません。多分、文字化けすると思います。

#### ソースコード

[こちらのサイト]("https://www.craneto.co.jp/archives/1219/")に掲載されているコードを使って、CREATE処理とINSERT処理を実行しています。ソースコードは、以下の通りです。

```python
import mysql.connector as mydb

# コネクションの作成
conn = mydb.connect(
    host='MySQL8',
    port='3306',
    user='root',
    password='passwd',
    database='test-mysql-python3'
)

# コネクションが切れた時に再接続してくれるよう設定
conn.ping(reconnect=True)

# 接続できているかどうか確認
print(conn.is_connected())

# DB操作用にカーソルを作成
cursor = conn.cursor()

# CREATE
# id, name だけのシンプルなテーブルを作成。id を主キーに設定。
cursor.execute("DROP TABLE IF EXISTS `sample`")
cursor.execute("""CREATE TABLE IF NOT EXISTS `sample` (
`id` int(11) NOT NULL,
`name` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL,
PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci""")

# INSERT
cursor.execute("INSERT INTO sample VALUES (1, 'sato')")

# プレースホルダの使用例
# 1つの場合には最後に , がないとエラー。('鈴木') ではなく ('鈴木',)
cursor.execute("INSERT INTO sample VALUES (2, %s)", ('suzuki',))
cursor.execute("INSERT INTO sample VALUES (%s, %s)", (3, 'takahashi'))
cursor.execute("INSERT INTO sample VALUES (%(id)s, %(name)s)", {'id': 4, 'name': 'tanaka'})

# 複数レコードを一度に挿入 executemany メソッドを使用
persons = [
    (5, 'ito'),
    (6, 'watanabe'),
]
cursor.executemany("INSERT INTO sample VALUES (%s, %s)", persons)

# DB操作が終わったらカーソルとコネクションを閉じる
cursor.close()
conn.commit()
conn.close()
```

#### 実行後の結果

上記コードを実行することで、MySQL8コンテナ上のDBに、テーブルが作成されているか確認してみます。

確認の際は、VSCode上から、MySQL8コンテナのターミナル(/bin/sh)にアタッチします。

[MySQL8のターミナルにアタッチ.png](MySQL8のターミナルにアタッチ.png)

アタッチ後、```mysql```を起動して、テーブルの内容を確認します。確認結果は、下図の通りです。

[テーブルの内容.png](テーブルの内容.png)

-----

## まとめ(感想)

公式ドキュメントを参考に環境構築してみましたが、形になるまで意外と時間がかかりました。

結構大変でしたが、VSCodeで、Pythonリモート開発環境を構築することができました。

-----

## 参考サイト

公式のドキュメントは、こちらからアクセスして下さい。

[https://code.visualstudio.com/docs/remote/containers](https://code.visualstudio.com/docs/remote/containers)






