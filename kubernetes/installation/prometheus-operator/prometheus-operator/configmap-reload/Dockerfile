FROM quay.io/coreos/configmap-reload:v0.0.1
MAINTAINER Li Sen  <lisen@qibeitech.com>
RUN rm -f /etc/localtime && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo 'Asia/Shanghai' >/etc/timezone && \
    echo "export LC_ALL=en_US.UTF-8" >> /etc/profile
