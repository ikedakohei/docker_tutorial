# Docker メモ

[こちら](https://www.udemy.com/course/docker-k/)のUdemyのコースをもとにとったメモ。

## Dockerコンテナの実行

### `docker run`コマンド

`docker run`コマンドは3つのコマンドに分割できる

- `docker pull` : イメージの取得
- `docker create` : コンテナの作成
- `docker start` : コンテナの起動

### Docker イメージ

- コンテナ実行に必要なファイルをまとめたファイルシステム
- イメージ上のデータはレイヤで構成された読み取り専用

### `docker run`実行時にコンテナ内で呼び出すコマンド

```shell
$ docker run docker/whalesay cowsay Hello!!
```

上記の `cowsay Hello!!` の部分がコンテナ内で呼び出すコマンド

### ローカル上にダウンロード済のイメージ一覧を表示するコマンド

```shell
$ docker images
```

### イメージにタグ付けするコマンド(任意のイメージ名でタグ付け)

```shell
$ docker tag docker/whalesay my_whalesay:ver1
```

`my_whalesay:ver1`という`docker/whalesay`のエイリアスができる(IMAGE IDが同じ)。

### ローカルのイメージを削除するコマンド

```shell
$ docker rmi docker/whalesay
```

強制削除の場合（コンテナごと削除）

```shell
$ docker rmi -f docker/whalesay
```

### `docker inspect`コマンド

イメージの詳細情報を表示するコマンド

```shell
$ docker inspect my_whalesay
```

### `docker pull`コマンド

イメージを取得するコマンド

```shell
$ docker pull docker/whalesay
```

### Dockerfileを使用したイメージビルド

Dockerfileの基本的な書き方

```dockerfile
# 元となるイメージの取得
FROM docker/whalesay:latest
# イメージビルド時に実行する命令
RUN apt-get -y update && apt-get install -y fortunes
# コンテナ起動時に実行する命令
CMD /usr/games/fortune | cowsay
```

Dockerfileからイメージをビルドするコマンド

```shell
$ docker build -t <イメージ名> <パス>
# 例
$ docker build -t docker-whale .
# キャッシュを無効にしたい場合
$ docker build --no-cache -t <イメージ名> <パス>
```

- `build` : イメージをビルドするサブコマンド
- `-t イメージ名` : タグ名の指定
- `パス` : ビルドコンテキスト(この場合は`Dockerfile`)のある場所を指定
- `--no-cache` : キャッシュを無効にする。例えば、`RUN apt-get -y update`などの記述がある場合に、一度実行（ビルド）済のDockerfileを使用しても、`apt-get -y update`は同じコマンドと判断されて実行されない。`--no-cache`オプションをつけることで、キャッシュが無効となり、実行されるようになる。

### Docker Hubへのイメージのプッシュ方法

```shell
# Docker Hubにログイン
$ docker login
# プッシュするためにイメージをタグ付け
$ docker tag docker-whale ikedako/docker-whale:ver1
# Docker Hubにプッシュ
$ docker push ikedako/docker-whale:ver1
```

### デタッチモード

```shell
$ docker run --name <コンテナ名> -d \
  -p <ホスト側のポート番号>:<コンテナ側のポート番号> \
  <イメージ名>
```

- `\` : 複数行でコマンドを記述するときに使用
- `--name <コンテナ名>` : 起動するコンテナに名前をつけるオプション
- `-d` :   デタッチモード。コンテナの実行をバックグラウンドで行う。
- `-p <ホスト側のポート番号>:<コンテナ側のポート番号>` : コンテナのポートをコンテナ外に開放する設定

### バインドマウント

バインドマウントによって、コンテナの外にあるデータを、コンテナの中で利用できる状態にすることができる。

```shell
$ docker run --name <コンテナ名> -d \
  -v <ホスト側のディレクトリ>:<コンテナ側のマウントポイント>:<ポイント>
  -p <ホスト側のポート番号>:<コンテナ側のポート番号> \
  <イメージ名>
```

- `-v <ホスト側のディレクトリ>:<コンテナ側のマウントポイント>:<オプション>`

  バインドマウント。オプションに`ro`が入ると、read onlyの意味で、読み取り専用でマウントする。引数のパスは絶対パスにすることに注意。

- -v オプション引数の例 :

  ```shell
  # Macの場合
  -v /Users/<ユーザ名>/workspace/udemy/docker_tutorial/html:/usr/share/nginx/html:ro
  # Windowsの場合
  -v /c/Users/<ユーザ名>/workspace/udemy/docker_tutorial/html:/usr/share/nginx/html:ro
  ```

### DockerfileのCOPY命令

docker cpコマンドについて

```shell
# ホストマシンのファイルをコンテナ内にコピーする場合
$ docker cp <ホスト上のコピーしたいファイルのパス> <コンテナ名 or ID>:<コピー先のパス>
# コンテナ内のファイルをホストマシンにコピーする場合
$ docker cp <コンテナ名 or ID>:<コンテナ上のコピーしたいファイルのパス> <ホスト上のコピー先のパス>
```

DockerfileのCOPY命令

```dockerfile
FROM nginx:latest
COPY default.conf /etc/nginx/conf.d/default.conf
```

上記の例ではホスト側にある`default.conf`ファイルをコンテナ側の`/etc/nginx/conf.d/default.conf`ファイルにコピーしている。

### `docker ps`コマンド

- 実行中のコンテナを確認する。
- 停止中のコンテナも確認するには、`docker ps -a`。

### Dockerコンテナのステータス

`docker ps -a`で確認できる。または、`docker inspect <コンテナ名 or ID>`

- `created` : 作られた状態 (`docker create`)
- `running` : 実行中 (`docker start`)
- `paused` : コンテナを一時停止 (`docker pause`。再開するには`docker unpause`)。
- `exited` : 停止中 (`docker stop`)
- `restarting` : 再起動中 (`docker create`)
- `dead` : コンテナが壊れた状態。`remove`するしかない。
- `removing` : コンテナ削除中 (`docker rm`)

### コンテナのシェルへの接続

- `docker attach <コンテナ名 or コンテナID>`

  ※ただし、シェルに接続できるのは、コンテナでシェルを実行している場合のみ

- `docker exec -it <コンテナ名 or コンテナID> /bin/bash`

  ※`/bin/bash`を実行しているが、bashがなければ別のシェルを指定する。

- `docker attach`で接続したシェルで`exit`を入力するとコンテナが停止する。

- `docker exec`で接続したシェルで`exit`を入力してもコンテナは停止しない。

- `-it` オプション

  - 下記のように使って、ホスト上で、コンテナ内のシェルを使用できるようにする。

  - ```shell
    $ docker run --name status-test -it alpine /bin/sh
    ```

- `-i` オプション
  - ホストの入力をコンテナの標準出力をつなげる
  - iはinteractiveの略。
  - 対話するようにシェルなどでコンテナ内で操作ができる
  
- `-t` オプション
  - コンテナの標準出力とホストの出力をつなげる
  - tty(端末デバイス)を割り当てている
  - -tはttyの略。

### Dockerコミット

```shell
$ docker commit <コンテナ名 or コンテナID> <イメージ名>:<タグ名>
```

- コンテナからイメージを作成する。
- `docker history <イメージ名 or イメージID>`でレイヤーを確認できるが、コンテナ内で実行したコマンドは履歴として残らない。

### linkオプションの使い方(※レガシー)

- レガシーな使い方のため、極力使わない方がよい。

- ```shell
  docker run --link <コンテナ名 or コンテナID>:<リンク先コンテナの別名> ...
  # 使用例
  docker run --name reverse-proxy -p 8080:8080 --link static-site:ss -d reverse-proxy
  ```

- コンテナ名、または別名(エイリアス)でリンク先に通信できるようになる。

- リンク先の環境変数や、リンク先コンテナのネットワークに関する環境変数が起動したコンテナに追加される。

### `-e` オプション

引数の環境変数を設定する。下記のような使い方をする。

```shell
$ docker run --name static-site -e AUTHOR="Kohei Ikeda" -d dockersamples/static-site
```

## Automated Build

### Automated Buildの設定

参考: https://qiita.com/Brutus/items/19f02df409e859406914

Automated Buildを設定することで、GitHubにDockerfileをpushすると、自動でイメージのビルドが行われる。

## Docker Machine

- Docker Engineを搭載した仮想マシンの作成、起動、停止、再起動などをコマンドラインから実行できるツール。
- ローカルPCだけでなく、リモートのクラウドプロバイダでDockerホストを立ち上げ管理することも可能。

### Docker for Windowsを使用している場合の注意点

`Hyper-V`が有効になっていると下記のようなエラーが起こる。

```shell
$ docker-machine create create-test
Creating CA: C:\Users\kouhei.ikeda\.docker\machine\certs\ca.pem
Creating client certificate: C:\Users\kouhei.ikeda\.docker\machine\certs\cert.pem
Running pre-create checks...
Error with pre-create check: "VBoxManage not found. Make sure VirtualBox is installed and VBoxManage is in the path"
You can further specify your shell with either 'cmd' or 'powershell' with the --shell flag.
```

エラーを回避するには、`--driver`を使用する。

`Hyper-Vマネージャー`で`仮想スイッチマネージャー`からを仮想スイッチ（外部）の作成をしてから、管理者権限で立ち上げたPowerShellで実行

```shell
$ docker-machine create --driver hyperv create-test
```

### Dokcer Machineを使用したDockerホストの管理

Dockerホストの一覧を確認する

```shell
$ docker-machine ls
NAME   ACTIVE   DRIVER   STATE   URL   SWARM   DOCKER   ERRORS
```

Dockerホストの作成

```shell
# Hyper-V(Windows)の場合
$ docker-machine create --driver hyperv <Dockerホスト名>
# VirtualBoxの場合
$ docker-machine create --driver virtualbox <Dockerホスト名>
```

操作対象のDockerホストを指定する(環境変数を設定する)ためのコマンドを調べる


```shell
# Windowsの場合
$ docker-machine env <Dockerホスト名> --shell powershell
# 下記を実行することで環境変数をまとめて設定できる
$ & "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env <Dockerホスト名> --shell powershell | Invoke-Expression

# Macの場合
$ docker-machine env <Dockerホスト名>
# 下記を実行することで環境変数をまとめて設定できる
$ eval $(docker-machine env <Dockerホスト名>)
```

環境変数を削除するためのコマンドを調べる

```shell
# Windowsの場合
$ docker-machine env -u --shell powershell
# 下記を実行することで設定した環境変数を削除できる
$ & "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env -u --shell powershell | Invoke-Expression

# Macの場合
$ docker-machine env -u
# 下記を実行することで設定した環境変数を削除できる
$ eval $(docker-machine env -u)
```



DockerホストにSSH接続する

```shell
$ docker ssh <Dockerホスト名>
```

Dockerホストのipアドレスを確認する

```shell
$ docker-machine ip <Dockerホスト名>
```

