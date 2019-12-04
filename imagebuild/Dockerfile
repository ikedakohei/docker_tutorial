# 元となるイメージの取得
FROM docker/whalesay:latest
# イメージビルド時に実行する命令
RUN apt-get -y update && apt-get install -y fortunes
# コンテナ起動時に実行する命令
CMD /usr/games/fortune | cowsay