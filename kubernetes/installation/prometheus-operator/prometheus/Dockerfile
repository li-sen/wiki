FROM quay.io/prometheus/prometheus:V2.3.2
MAINTAINER Li Sen  <lisen@qibeitech.com>
USER root
RUN rm -f /etc/localtime && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo 'Asia/Shanghai' >/etc/timezone && \
    echo "export LC_ALL=en_US.UTF-8" >> /etc/profile && \
    . /etc/profile
