FROM redis
# 设置环境变量，所有操作都是非交互式的
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get -y update \
  && apt-get -y upgrade \
  && apt-get -y --no-install-recommends install ruby wget \
  && gem install redis -v 3.3.5 \
  && apt-get -y autoremove \
  && apt-get -y clean 
# gem install redis -v 3.3.5 安装ruby-redis插件，插件最新版本查看：https://rubygems.org/gems/redis/versions
RUN wget -O /usr/local/bin/redis-trib http://download.redis.io/redis-stable/src/redis-trib.rb
RUN chmod 755 /usr/local/bin/redis-trib
CMD redis-server
