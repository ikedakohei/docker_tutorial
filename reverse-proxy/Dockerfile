FROM nginx:latest
COPY /reverse_proxy.conf /etc/nginx/conf.d/reverse_proxy.conf
RUN apt-get update && apt-get install -y inetutils-ping
