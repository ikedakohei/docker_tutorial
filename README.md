# Docker メモ

[こちら](https://www.udemy.com/course/docker-k/)のUdemyのコースをもとにとったメモ。

## Dockerコンテナの実行

### `docker run`コマンド

例:

```shell
$ docker run hello-world
```

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
  -v <ホスト側のディレクトリ>:<コンテナ側のマウントポイント>:<オプション>
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

- `docker network`を使うべき。

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

Dockerホストの起動

```shell
$ docker-machine start <Docker ホスト名>
```

Dockerホストの停止

```shell
$ docker-machine stop <Docker ホスト名>
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

Dockerホストの削除

```shell
$ docker-machine rm <Dockerホスト名>
```

DockerホストにSSH接続する

```shell
$ docker ssh <Dockerホスト名>
```

Dockerホストのipアドレスを確認する

```shell
$ docker-machine ip <Dockerホスト名>
```

AWSやGCPへのDockerホストのプロビジョニングもできる

## Dockerのネットワーク

参考: https://qiita.com/TsutomuNakamura/items/ed046ee21caca4a2ffd9

### デフォルトのブリッジネットワークとユーザー定義のブリッジネットワーク

`docker network ls`コマンドでネットワーク一覧を確認できる。Dockerがインストールされた時点で`bridge`、`host`、 `null`の3つのDRIVERが存在する。

```shell
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
7c91220232cd        bridge              bridge              local
781cea8d8cdb        host                host                local
336ae8279a1d        none                null                local
```

### bridge

ネットワークの詳細については下記で確認できる。

```shell
$ docker network inspect <ネットワーク名 or ネットワークID>
# 例
$ docker network inspect bridge
```

試しにコンテナを追加してみて、attachでコンテナ内に入って、ipアドレスを確認してみる。

```shell
$ docker run -itd --name alpine1 alpine /bin/sh
$ docker attach alpine1
# コンテナ内に入った
$ ip addr show
# ...省略...
inet 172.17.0.2/16
```

もう一つコンテナを追加してさきほど作成した`alpine1`にpingを送ってみると通信できていることが確認できる。

```shell
$ docker run -itd --name alpine2 alpine /bin/sh
$ docker attach alpine2
# コンテナ内に入った
$ ip addr show
...省略...
inet 172.17.0.3/16
# alpin1にpingを送る
$ ping -w 3 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.539 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.175 ms
64 bytes from 172.17.0.2: seq=2 ttl=64 time=0.188 ms
# コンテナ名でalpine1にpingを送ると失敗する
$ ping -w 3 alpine1
ping: bad address 'alpine1'
```

コンテナ名で接続するためには、ユーザー定義のブリッジネットワークを作成する必要がある。

### ユーザー定義のブリッジネットワーク

#### ユーザー定義のブリッジネットワークの作成方法

```shell
$ docker network create <ネットワーク名>
```

例：

```shell
$ docker network create my_nw
# my_nwという新しいブリッジネットワークが作成されているのを確認
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
7c91220232cd        bridge              bridge              local
781cea8d8cdb        host                host                local
d0b7d46a90a9        my_nw               bridge              local
336ae8279a1d        none                null                local
```

#### ユーザー定義のブリッジネットワークとコンテナの接続方法

```shell
$ docker network connect <ネットワーク名> <コンテナ名>
```

例：

```shell
# my_nwというネットワークにalpine1コンテナを接続
$ docker network connect my_nw alpine1
# my_nwというネットワークにalpine2コンテナを接続
$ docker network connect my_nw alpine2
```

#### コンテナ作成時にユーザー定義のネットワークと接続する方法

```shell
$ docker run --name <コンテナ名> --network <ネットワーク名> <イメージ名>
```

例:

```shell
$ docker run -itd --name alpine3 --network my_nw alpine
```

#### コンテナ名で接続できるか確認

```shell
$ docker network inspect my_nw
...省略...
"Containers": {
    "61909d8e24e1a42e7d48e769827f1228e8e4af92d63efa47af210c96dafc0185": {
        "Name": "alpine2",
        "EndpointID": "4f3eb4f48ae03e6cab10e812938a2a6d692f1694d9e77b980e9cd21873172403",
        "MacAddress": "02:42:ac:12:00:04",
        "IPv4Address": "172.18.0.4/16",
        "IPv6Address": ""
    },
    "6856d78b1d64dd9f92e285e19b5ec21cf0ed2e468a1da9547b78fc6cb6aca681": {
        "Name": "alpine1",
        "EndpointID": "e8b85c05206a6ea1c882cb462bbc9add6139ca992a6ea82db7ca92abf3c33852",
        "MacAddress": "02:42:ac:12:00:03",
        "IPv4Address": "172.18.0.3/16",
        "IPv6Address": ""
    },
    "f60f180ec5f296e5d93f8a2761f87309096a8a659e0cd7ed9c3231ab9975d7a3": {
        "Name": "alpine3",
        "EndpointID": "0cdbc0c59fdb801200e304347615e672a609dd407f6545d3119931125de9508d",
        "MacAddress": "02:42:ac:12:00:02",
        "IPv4Address": "172.18.0.2/16",
        "IPv6Address": ""
    }
}
...省略...

$ docker attach alpine2
# コンテナ内に入った
$ ping -w 3 alpine1
PING alpine1 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.051 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.069 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.159 ms
$ ping -w 3 alpine3
PING alpine3 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.082 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.080 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.226 ms
```

コンテナ名でネットワーク接続できるようになった。

#### ネットワークからコンテナを切断する方法

```shell
$ docker network disconnect <ネットワーク名> <コンテナ名>
```

コンテナのネットワーク状況を確認

```shell
$ docker inspect alpine2
...省略...
"Networks": {
    "bridge": {
        "IPAMConfig": null,
        "Links": null,
        "Aliases": null,
        "NetworkID": "7c91220232cdc4bdd644c80f936372947617090a08393562edba81268687b2ce",
        "EndpointID": "b2d746e4550a2bebdb263a8ec6f5c08ce8a5754d676557d0c122ccbc1f814d66",
        "Gateway": "172.17.0.1",
        "IPAddress": "172.17.0.3",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "MacAddress": "02:42:ac:11:00:03",
        "DriverOpts": null
    },
    "my_nw": {
        "IPAMConfig": {},
        "Links": null,
        "Aliases": [
            "61909d8e24e1"
        ],
        "NetworkID": "d0b7d46a90a9c0f2b606d786fd9a3e4148b05b411334db6302b65480bd825ace",
        "EndpointID": "25e20b7739a6534612d3a9aa2424720e0b0447b820c17d121133977c3df1b08c",
        "Gateway": "172.18.0.1",
        "IPAddress": "172.18.0.4",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "MacAddress": "02:42:ac:12:00:04",
        "DriverOpts": {}
    }
}
```

`bridge`と`my_nw`というふたつのネットワークに接続していることがわかる。

`alpine2`コンテナから、デフォルトの`bridge`ネットワークを切断する。

```shell
$ docker network disconnect bridge alpine2
# 切断できたか確認
$ docker inspect alpine2
...省略...
"Networks": {
    "my_nw": {
        "IPAMConfig": {},
        "Links": null,
        "Aliases": [
            "61909d8e24e1"
        ],
        "NetworkID": "d0b7d46a90a9c0f2b606d786fd9a3e4148b05b411334db6302b65480bd825ace",
        "EndpointID": "25e20b7739a6534612d3a9aa2424720e0b0447b820c17d121133977c3df1b08c",
        "Gateway": "172.18.0.1",
        "IPAddress": "172.18.0.4",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "MacAddress": "02:42:ac:12:00:04",
        "DriverOpts": {}
    }
}
```

### noneネットワーク

noneネットワークはnullなネットワークドライバの実装。ネットワーク接続を必要としないコンテナを作成する場合に使用される。

noneネットワークに接続したコンテナを作成

```shell
$ docker run -itd --name none --network none alpine /bin/sh
```

ネットワーク構成の確認

```shell
$ docker inspect none
...省略...
"Networks": {
    "none": {
        "IPAMConfig": null,
        "Links": null,
        "Aliases": null,
        "NetworkID": "336ae8279a1da8c2653b5a4d9e8596b38c72a115a0af8e2c7293d2a39cb96972",
        "EndpointID": "94864ab640f6afaf9e4d5a84b9143d47dada176b22ed9e9ad321a07ff11ce8e3",
        "Gateway": "",
        "IPAddress": "",
        "IPPrefixLen": 0,
        "IPv6Gateway": "",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "MacAddress": "",
        "DriverOpts": null
    }
}
```

noneコンテナにアタッチしてネットワーク構成の確認

```shell
$ docker attach none
# コンテナ内に入った
$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
```

ループバックインターフェース以外にネットワークインターフェースを持たないことが確認できた。

### hostネットワーク

hostネットワークは、Dockerホストと同じネットワークにスタックするドライバで、Dockerホストマシンと同じネットワークインタフェース、IPアドレスを持つようになる。

DockerホストのIPを確認

```shell
$ docker-machine ip nw-vm1
192.168.232.157
```

```shell
# ssh接続
$ docker-machine ssh nw-vm1
# コンテナ作成(-pフラグを使用していないことに注目)
$ docker run -d --name web --network host nginx
```

`192.168.232.157`にアクセスするとnginxのデフォルトページが表示される。

`-p`フラグを使用してポートを開放する必要がない。

## Dockerのデータ管理

3種類のデータ管理方法がある

- volume
- bind mount
- tmpfs

### volumeの使い方

#### volumeを使うテスト環境のためにホスト作成

```shell
$ docker-machine create --driver hyperv vol-test
```

#### 作成したホスト環境にssh接続

```shell
$ docker-machine ssh vol-test
```

#### volumeを作成する

```shell
$ docker volume create my-vol
```

#### 作成されたvolumeを確認する

```shell
$ docker volume ls
DRIVER              VOLUME NAME
local               my-vol
```

#### 作成したvolumeの詳細を調べる

```shell
$ docker volume inspect my-vol
[
    {
        "CreatedAt": "2019-12-09T02:33:02Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/mnt/sda1/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
```

#### volumeを削除する

```shell
$ docker volume rm my-vol
```

#### コンテナにvolumeをマウントする方法

##### -vオプション

```shell
$ docker run -itd --name mount-c1 -v vol1:/app nginx:latest
```

`-v vol1:/app` : `vol1`というvolumeが存在しない場合、新規に`vol1`というvolumeが作成される。コンテナ内のマウントポイントは`/app`。

コンテナ情報を確認

```shell
$ docker inspect mount-c1
...省略...
"Mounts": [
    {
        "Type": "volume",
        "Name": "vol1",
        "Source": "/mnt/sda1/var/lib/docker/volumes/vol1/_data",
        "Destination": "/app",
        "Driver": "local",
        "Mode": "z",
        "RW": true,
        "Propagation": ""
    }
]
```

mount-c1コンテナ内に入って、マウントした`/app`配下にファイルを作成してみる。

```shell
$ docker exec -it mount-c1 /bin/bash
# mount-c1コンテナに入った
$ df
Filesystem     1K-blocks   Used Available Use% Mounted on
overlay         18714000 181948  17542924   2% /
tmpfs              65536      0     65536   0% /dev
tmpfs             473632      0    473632   0% /sys/fs/cgroup
shm                65536      0     65536   0% /dev/shm
# /appが一つのファイルシステムとしてマウントされている
/dev/sda1       18714000 181948  17542924   2% /app
tmpfs             473632      0    473632   0% /proc/asound
tmpfs             473632      0    473632   0% /proc/acpi
tmpfs             473632      0    473632   0% /proc/scsi
tmpfs             473632      0    473632   0% /sys/firmware
# /app配下に適当なファイルを作成してみる
$ touch /app/hogehoge
```

別のコンテナを`vol1`をマウント元として作成する。hogehogeファイルが存在しているか確認。

```shell
$ docker run -itd --name mount-c2 --mount source=vol1,target=/app nginx:latest
```

`--mount source=vol1,target=/app` : マウント元のvolumeをsourceに指定、マウント先を/appに指定。

volumeは同じホスト内でないと使用できないことに注意。

#### コンテナ起動時のvolumeマウント

コンテナ起動時にvolumeを作成してマウントすると、マウント先(コンテナ内)のファイルがvolume内にコピーされる。

`--mount source=copy-vol,destination=/etc/nginx ` のような使い方のほうが`-v`よりも推奨されている。

```shell
# copy-volというvolumeが存在しないため、copy-volというvolumeが作成された
$ docker run -itd --name mount-c3 --mount source=copy-vol,destination=/etc/nginx nginx
# copy-volの情報を確認
$ docker volume inspect copy-vol
[
    {
        "CreatedAt": "2019-12-09T06:49:48Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/mnt/sda1/var/lib/docker/volumes/copy-vol/_data",
        "Name": "copy-vol",
        "Options": null,
        "Scope": "local"
    }
]
# Mountpointのディレクトリの内容を確認
# コンテナ側にあったファイルが格納されていることが確認できる
$ sudo ls -la /var/lib/docker/volumes/copy-vol/_data
total 48
drwxr-xr-x    3 root     root          4096 Dec  9 06:49 .
drwxr-xr-x    3 root     root          4096 Dec  9 06:49 ..
drwxr-xr-x    2 root     root          4096 Dec  9 06:49 conf.d
-rw-r--r--    1 root     root          1007 Nov 19 12:50 fastcgi_params
-rw-r--r--    1 root     root          2837 Nov 19 12:50 koi-utf
-rw-r--r--    1 root     root          2223 Nov 19 12:50 koi-win
-rw-r--r--    1 root     root          5231 Nov 19 12:50 mime.types
lrwxrwxrwx    1 root     root            22 Nov 19 12:50 modules -> /usr/lib/nginx/modules
-rw-r--r--    1 root     root           643 Nov 19 12:50 nginx.conf
-rw-r--r--    1 root     root           636 Nov 19 12:50 scgi_params
-rw-r--r--    1 root     root           664 Nov 19 12:50 uwsgi_params
-rw-r--r--    1 root     root          3610 Nov 19 12:50 win-utf
```

#### 読み取り専用でマウントする場合

`--mount`を使用する場合

```shell
$ docker run -itd --name mount-c4 --mount source=copy-vol,destination=/etc/nginx,readonly nginx
$ docker inspect mount-c4
...省略...
"Mounts": [
    {
        "Type": "volume",
        "Name": "copy-vol",
        "Source": "/mnt/sda1/var/lib/docker/volumes/copy-vol/_data",
        "Destination": "/etc/nginx",
        "Driver": "local",
        "Mode": "z",
        # "RW"がfalseになっている
        "RW": false,
        "Propagation": ""
    }
]
```

`-v`を使用する場合

```shell
$ docker run -itd --name mount-c5 -v copy-vol:/etc/nginx:ro nginx
$ docker inspect mount-c4
...省略...
"Mounts": [
    {
        "Type": "volume",
        "Name": "copy-vol",
        "Source": "/mnt/sda1/var/lib/docker/volumes/copy-vol/_data",
        "Destination": "/etc/nginx",
        "Driver": "local",
        "Mode": "ro",
        # "RW"がfalseになっている
        "RW": false,
        "Propagation": ""
    }
]
```

### bind mountの使い方

`-v`でカレントディレクトリに存在しないディレクトリをマウントする。

```shell
$ docker run -itd --name bind-test1 -v "$(pwd)"/source:/app nginx
```

`--mount`でカレントディレクトリに存在しないディレクトリをマウントする(エラーになる)。※`type=bind`と明示する必要がある。

```shell
$ docker run -itd --name bind-test2 --mount type=bind,source="$(pwd)"/source2,target=/app nginx
# "$(pwd)"/source2が存在しないため、エラーが起こる
docker: Error response from daemon: invalid mount config for type "bind": bind source path does not exist: /home/docker/source2.
See 'docker run --help'.
```

bind mountしたときのコンテナの状態を確認

```shell
$ docker inspect bind-test1
...省略...
 "Mounts": [
    {
        # Typeがbindになっている
        "Type": "bind",
        # マウント元のディレクトリ
        "Source": "/home/docker/source",
        # マウント先のディレクトリ
        "Destination": "/app",
        "Mode": "",
        # デフォルトだと読み書き可能な状態
        "RW": true,
        "Propagation": "rprivate"
    }
]
```

bind mountは、コンテナ側の既存ファイルをなくしてしまう可能性などがあるため、使い方に注意する。

### tmpfs

ホストのメモリ上にマウントする。ホストかコンテナが停止するとデータは開放され、tmpfsマウントが取り除かれる。

`type=tmpfs`で指定する。

```shell
$ docker run -itd --name tmptest --mount type=tmpfs,destination=/app nginx
# オプションをいろいろつけられる
$ docker run -itd --name tmptest2 --mount type=tmpfs,destination=/app,tmpfs-size=500000000,tmpfs-mode=700 nginx
```

## Docker Compose

Compose実行のステップ

1. Dockerfileを用意するか、使用するイメージをDocker Hubなどに用意する
2. docker-compose.ymlを定義する
3. docker-compose upを実行する

docker-compose.ymlの例

```docker
# docker-cmpose.ymlの例
# docker-composeの形式のバージョン
version: '3'
services:
  # サービス名（任意の名前をつけられる）
  web:
    # webサービス用のDockerfileが同じディレクトリにある
    build: .
    # コンテナ外部に公開するポートとマッピング先のポート
    ports:
    - "5000:5000"
    # マウント
    volumes:
    # bind mount
    - .:/code
    # volume mount (volumesで定義している)
    - logvolume01:/var/log
    links
    - redis
  # サービス名（任意の名前をつけられる）
  redis:
    # imageをそのまま使用
    image: redis
# volumeの定義
volumes:
  logvolume01: {}
```

