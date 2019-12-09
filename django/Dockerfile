# イメージ名
FROM python:3
# 環境変数の設定
ENV PYTHONBUFFERED 1
# /codeディレクトリを作成
RUN mkdir /code
# WORKDIRはDockerfileでRUN、CMD、ENTRYPOINT、COPY、ADD命令実行時の作業ディレクトリを指定する
WORKDIR /code
# ホスト側にあるrequirements.txtをコンテナ側の/code/配下にコピー
COPY requirements.txt /code/
RUN pip install -r requirements.txt
# ビルドコンテキスト(Dockerfile)のあるディレクトリをcodeディレクトリ配下にコピー
COPY . /code/
