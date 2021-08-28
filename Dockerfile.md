
****
<details><summary>展开</summary><pre><code>

``` yaml

```
</code></pre></details>

----

****
<details><summary>展开</summary><pre><code>

``` yaml

```
</code></pre></details>

----

**https://github.com/nacos-group/nacos-docker/blob/master/build/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
FROM centos:7.5.1804
MAINTAINER pader "huangmnlove@163.com"

# set environment
ENV MODE="cluster" \
    PREFER_HOST_MODE="ip"\
    BASE_DIR="/home/nacos" \
    CLASSPATH=".:/home/nacos/conf:$CLASSPATH" \
    CLUSTER_CONF="/home/nacos/conf/cluster.conf" \
    FUNCTION_MODE="all" \
    JAVA_HOME="/usr/lib/jvm/java-1.8.0-openjdk" \
    NACOS_USER="nacos" \
    JAVA="/usr/lib/jvm/java-1.8.0-openjdk/bin/java" \
    JVM_XMS="1g" \
    JVM_XMX="1g" \
    JVM_XMN="512m" \
    JVM_MS="128m" \
    JVM_MMS="320m" \
    NACOS_DEBUG="n" \
    TOMCAT_ACCESSLOG_ENABLED="false" \
    TIME_ZONE="Asia/Shanghai"

ARG NACOS_VERSION=2.0.3
ARG HOT_FIX_FLAG=""

WORKDIR $BASE_DIR

RUN set -x \
    && yum update -y \
    && yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel wget iputils nc  vim libcurl
RUN wget  https://github.com/alibaba/nacos/releases/download/${NACOS_VERSION}${HOT_FIX_FLAG}/nacos-server-${NACOS_VERSION}.tar.gz -P /home
RUN tar -xzvf /home/nacos-server-${NACOS_VERSION}.tar.gz -C /home \
    && rm -rf /home/nacos-server-${NACOS_VERSION}.tar.gz /home/nacos/bin/* /home/nacos/conf/*.properties /home/nacos/conf/*.example /home/nacos/conf/nacos-mysql.sql
RUN yum autoremove -y wget \
    && ln -snf /usr/share/zoneinfo/$TIME_ZONE /etc/localtime && echo $TIME_ZONE > /etc/timezone \
    && yum clean all

ADD bin/docker-startup.sh bin/docker-startup.sh
ADD conf/application.properties conf/application.properties

# set startup log dir
RUN mkdir -p logs \
	&& cd logs \
	&& touch start.out \
	&& ln -sf /dev/stdout start.out \
	&& ln -sf /dev/stderr start.out
RUN chmod +x bin/docker-startup.sh

EXPOSE 8848
ENTRYPOINT ["bin/docker-startup.sh"]
```
</code></pre></details>

----

**https://github.com/CybexDex/seed/blob/master/buildimage/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
FROM ubuntu:16.04

RUN apt-get update && apt-get install -y vim expect

ENV PORT 8899

ARG VERSION
ARG BUILD_DATE

WORKDIR	/usr/app

VOLUME /usr/app

COPY . .

LABEL bn.version=$VERSION
LABEL bn.build_date=$BUILD_DATE
LABEL bn.build_cmd="docker build --force-rm --no-cache --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') --build-arg VERSION=$VERSION -t jadepool-seed:$VERSION -f ./Dockerfile ."
LABEL bn.run_cmd="docker run -d --name jadepool-seed -p 8899:8899 -v /data/seed-data:/usr/app jadepool-seed:$VERSION"

EXPOSE $PORT

CMD ./startup.sh $PORT
```
</code></pre></details>

----

**https://github.com/drone/drone/blob/master/docker/Dockerfile.agent.linux.amd64**
<details><summary>展开</summary><pre><code>

``` yaml
  
FROM alpine:3.11 as alpine
RUN apk add -U --no-cache ca-certificates

FROM alpine:3.11
ENV GODEBUG netdns=go
ENV DRONE_RUNNER_OS=linux
ENV DRONE_RUNNER_ARCH=amd64
ENV DRONE_RUNNER_PLATFORM=linux/amd64
ENV DRONE_RUNNER_CAPACITY=1
ADD release/linux/amd64/drone-agent /bin/

RUN [ ! -e /etc/nsswitch.conf ] && echo 'hosts: files dns' > /etc/nsswitch.conf

COPY --from=alpine /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

LABEL com.centurylinklabs.watchtower.stop-signal="SIGINT"

ENTRYPOINT ["/bin/drone-agent"]
```
</code></pre></details>

----

**https://github.com/drone/drone/blob/master/docker/Dockerfile.server.linux.amd64**
<details><summary>展开</summary><pre><code>

``` yaml
# docker build --rm -f docker/Dockerfile -t drone/drone .

FROM alpine:3.11 as alpine
RUN apk add -U --no-cache ca-certificates

FROM alpine:3.11
EXPOSE 80 443
VOLUME /data

RUN [ ! -e /etc/nsswitch.conf ] && echo 'hosts: files dns' > /etc/nsswitch.conf

ENV GODEBUG netdns=go
ENV XDG_CACHE_HOME /data
ENV DRONE_DATABASE_DRIVER sqlite3
ENV DRONE_DATABASE_DATASOURCE /data/database.sqlite
ENV DRONE_RUNNER_OS=linux
ENV DRONE_RUNNER_ARCH=amd64
ENV DRONE_SERVER_PORT=:80
ENV DRONE_SERVER_HOST=localhost
ENV DRONE_DATADOG_ENABLED=true
ENV DRONE_DATADOG_ENDPOINT=https://stats.drone.ci/api/v1/series

COPY --from=alpine /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

ADD release/linux/amd64/drone-server /bin/
ENTRYPOINT ["/bin/drone-server"]
```
</code></pre></details>

----

**https://github.com/jumpserver/jumpserver/blob/master/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
# 编译代码
FROM python:3.8.6-slim as stage-build
MAINTAINER JumpServer Team <ibuler@qq.com>
ARG VERSION
ENV VERSION=$VERSION

WORKDIR /opt/jumpserver
ADD . .
RUN cd utils && bash -ixeu build.sh


# 构建运行时环境
FROM python:3.8.6-slim
ARG PIP_MIRROR=https://pypi.douban.com/simple
ENV PIP_MIRROR=$PIP_MIRROR
ARG PIP_JMS_MIRROR=https://pypi.douban.com/simple
ENV PIP_JMS_MIRROR=$PIP_JMS_MIRROR

WORKDIR /opt/jumpserver

COPY ./requirements/deb_buster_requirements.txt ./requirements/deb_buster_requirements.txt
RUN sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list \
    && sed -i 's/security.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list \
    && apt update \
    && grep -v '^#' ./requirements/deb_buster_requirements.txt | xargs apt -y install \
    && rm -rf /var/lib/apt/lists/* \
    && localedef -c -f UTF-8 -i zh_CN zh_CN.UTF-8 \
    && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

COPY ./requirements/requirements.txt ./requirements/requirements.txt
RUN pip install --upgrade pip==20.2.4 setuptools==49.6.0 wheel==0.34.2 -i ${PIP_MIRROR} \
    && pip config set global.index-url ${PIP_MIRROR} \
    && pip install --no-cache-dir $(grep 'jms' requirements/requirements.txt) -i ${PIP_JMS_MIRROR} \
    && pip install --no-cache-dir -r requirements/requirements.txt

COPY --from=stage-build /opt/jumpserver/release/jumpserver /opt/jumpserver
RUN mkdir -p /root/.ssh/ \
    && echo "Host *\n\tStrictHostKeyChecking no\n\tUserKnownHostsFile /dev/null" > /root/.ssh/config

RUN echo > config.yml
VOLUME /opt/jumpserver/data
VOLUME /opt/jumpserver/logs

ENV LANG=zh_CN.UTF-8

EXPOSE 8070
EXPOSE 8080
ENTRYPOINT ["./entrypoint.sh"]
```
</code></pre></details>

----

**https://github.com/hhyo/Archery/blob/master/src/docker/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
FROM hhyo/archery-base:1.3

WORKDIR /opt/archery

COPY . /opt/archery/

#archery
RUN cd /opt \
    && yum -y install nginx \
    && source /opt/venv4archery/bin/activate \
    && pip3 install -r /opt/archery/requirements.txt \
    && cp /opt/archery/src/docker/nginx.conf /etc/nginx/ \
    && cp /opt/archery/src/docker/supervisord.conf /etc/ \
    && mv /opt/sqladvisor /opt/archery/src/plugins/ \
    && mv /opt/soar /opt/archery/src/plugins/ \
    && mv /opt/tmp_binlog2sql /opt/archery/src/plugins/binlog2sql

#port
EXPOSE 9123

#start service
ENTRYPOINT bash /opt/archery/src/docker/startup.sh && bash
```
</code></pre></details>

----

**https://github.com/hhyo/Archery/blob/master/src/docker/Dockerfile-base**
<details><summary>展开</summary><pre><code>

``` yaml
FROM docker.io/centos:7

ENV PYTHON_VERSION 3.8.6
ENV DOCKERIZE_VERSION v0.6.1
ENV SOAR_VERSION 0.11.0

ENV TZ=Asia/Shanghai
ENV LANG en_US.UTF-8

WORKDIR /opt

#locale
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone \
    && yum -y install kde-l10n-Chinese \
    && localedef -c -f UTF-8 -i zh_CN zh_CN.utf8

ENV LC_ALL zh_CN.utf8

#python
RUN yum -y install libffi-devel wget gcc make zlib-devel openssl openssl-devel ncurses-devel openldap-devel gettext \
    && cd /opt \
    && wget "https://www.python.org/ftp/python/$PYTHON_VERSION/Python-$PYTHON_VERSION.tar.xz" \
    && tar -xvJf Python-$PYTHON_VERSION.tar.xz \
    && cd /opt/Python-$PYTHON_VERSION \
    && ./configure prefix=/usr/local/python3 \
    && make && make install \
    && ln -fs /usr/local/python3/bin/python3 /usr/bin/python3 \
    && ln -fs /usr/local/python3/bin/pip3 /usr/bin/pip3 \
    && pip3 install virtualenv \
    && cd /opt \
    && ln -fs /usr/local/python3/bin/virtualenv /usr/bin/virtualenv \
    && virtualenv venv4archery --python=python3 \
    && rm -rf Python-$PYTHON_VERSION \
    && rm -rf Python-$PYTHON_VERSION.tar.xz \
#dockerize
    && wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
#sqladvisor
    && yum -y install epel-release \
    && yum -y install cmake bison gcc-c++ git mysql-devel libaio-devel  glib2 glib2-devel \
    && yum -y install https://repo.percona.com/yum/percona-release-latest.noarch.rpm \
    && yum -y install Percona-Server-devel-56 Percona-Server-shared-56  Percona-Server-client-56 \
    && yum -y install percona-toolkit \
    && cd /opt \
    && git clone https://github.com/hhyo/SQLAdvisor.git --depth 3 \
    && cd /opt/SQLAdvisor/ \
    && cmake -DBUILD_CONFIG=mysql_release -DCMAKE_BUILD_TYPE=debug -DCMAKE_INSTALL_PREFIX=/usr/local/sqlparser ./ \
    && make && make install \
    && cd sqladvisor/ \
    && cmake -DCMAKE_BUILD_TYPE=debug ./ \
    && make \
    && mv /opt/SQLAdvisor/sqladvisor/sqladvisor /opt \
    && rm -rf /opt/SQLAdvisor/
#soar
RUN cd /opt \
    && wget https://github.com/XiaoMi/soar/releases/download/$SOAR_VERSION/soar.linux-amd64 -O soar \
    && chmod a+x soar \
#binlog2sql
    && cd /opt \
    && git clone https://github.com/danfengcao/binlog2sql.git \
    && mv binlog2sql/binlog2sql/ tmp_binlog2sql \
    && rm -rf binlog2sql \
#msodbc
    && cd /opt \
    && curl https://packages.microsoft.com/config/rhel/7/prod.repo > /etc/yum.repos.d/mssql-release.repo \
    && ACCEPT_EULA=Y yum -y install msodbcsql17 \
    && yum -y install unixODBC-devel \
#oracle instantclient
    && yum -y install http://yum.oracle.com/repo/OracleLinux/OL7/oracle/instantclient/x86_64/getPackage/oracle-instantclient19.3-basiclite-19.3.0.0.0-1.x86_64.rpm \
#mongo instantclient
    && wget -c https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.6.20.tgz \
    && tar -xvf mongodb-linux-x86_64-rhel70-3.6.20.tgz \
    && cp mongodb-linux-x86_64-rhel70-3.6.20/bin/mongo /usr/local/bin/mongo \
    && rm -rf mongodb-linux-x86_64-*
```
</code></pre></details>

----

**https://github.com/nirui/sshwifty/blob/master/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
# Build the build base environment
FROM debian:testing AS base
RUN set -ex && \
    cd / && \
    echo '#!/bin/sh' > /try.sh && echo 'res=0; for i in $(seq 0 36); do $@; res=$?; [ $res -eq 0 ] && exit $res || sleep 10; done; exit $res' >> /try.sh && chmod +x /try.sh && \
    echo '#!/bin/sh' > /child.sh && echo 'cpid=""; ret=0; i=0; for c in "$@"; do ( (((((eval "$c"; echo $? >&3) | sed "s/^/|-($i) /" >&4) 2>&1 | sed "s/^/|-($i)!/" >&2) 3>&1) | (read xs; exit $xs)) 4>&1) & ppid=$!; cpid="$cpid $ppid"; echo "+ Child $i (PID $ppid): $c ..."; i=$((i+1)); done; for c in $cpid; do wait $c; cret=$?; [ $cret -eq 0 ] && continue; echo "* Child PID $c has failed." >&2; ret=$cret; done; exit $ret' >> /child.sh && chmod +x /child.sh && \
    export PATH=$PATH:/ && \
    ([ -z "$HTTP_PROXY" ] || (echo "Acquire::http::Proxy \"$HTTP_PROXY\";" >> /etc/apt/apt.conf)) && \
    ([ -z "$HTTPS_PROXY" ] || (echo "Acquire::https::Proxy \"$HTTPS_PROXY\";" >> /etc/apt/apt.conf)) && \
    (echo "Acquire::Retries \"8\";" >> /etc/apt/apt.conf) && \
    echo '#!/bin/sh' > /install.sh && echo 'apt-get update && apt-get install autoconf automake libtool build-essential ca-certificates curl git npm golang-go libvips libvips-dev -y' >> /install.sh && chmod +x /install.sh && \
    /try.sh /install.sh && rm /install.sh && \
    /try.sh update-ca-certificates -f && c_rehash && \
    ([ -z "$HTTP_PROXY" ] || (git config --global http.proxy "$HTTP_PROXY" && npm config set proxy "$HTTP_PROXY")) && \
    ([ -z "$HTTPS_PROXY" ] || (git config --global https.proxy "$HTTPS_PROXY" && npm config set https-proxy "$HTTPS_PROXY")) && \
    export PATH=$PATH:"$(go env GOPATH)/bin" && \
    ([ -z "$CUSTOM_COMMAND" ] || (echo "Running custom command: $CUSTOM_COMMAND" && $CUSTOM_COMMAND)) && \
    echo '#!/bin/sh' > /install.sh && echo "npm install -g npm || (npm cache clean -f && false)" >> /install.sh && chmod +x /install.sh && /try.sh /install.sh && rm /install.sh

# Build the base environment for application libraries
FROM base AS libbase
COPY . /tmp/.build/sshwifty
RUN set -ex && \
    cd / && \
    export PATH=$PATH:/ && \
    export CPPFLAGS='-DPNG_ARM_NEON_OPT=0' && \
    /try.sh apt-get install libpng-dev -y && \
    ls -l /tmp/.build/sshwifty && \
    /child.sh \
        "cd /tmp/.build/sshwifty && echo '#!/bin/sh' > /npm_install.sh && echo \"npm install || (npm cache clean -f && rm ~/.npm/_* -rf && false)\" >> /npm_install.sh && chmod +x /npm_install.sh && /try.sh /npm_install.sh && rm /npm_install.sh" \
        'cd /tmp/.build/sshwifty && /try.sh go mod download'

# Main building environment
FROM libbase AS builder
RUN set -ex && \
    cd / && \
    export PATH=$PATH:/ && \
    ([ -z "$HTTP_PROXY" ] || (git config --global http.proxy "$HTTP_PROXY" && npm config set proxy "$HTTP_PROXY")) && \
    ([ -z "$HTTPS_PROXY" ] || (git config --global https.proxy "$HTTPS_PROXY" && npm config set https-proxy "$HTTPS_PROXY")) && \
    (cd /tmp/.build/sshwifty && /try.sh npm run build && mv ./sshwifty /)

# Build the final image for running
FROM alpine:latest
ENV SSHWIFTY_HOSTNAME= \
    SSHWIFTY_SHAREDKEY= \
    SSHWIFTY_DIALTIMEOUT=10 \
    SSHWIFTY_SOCKS5= \
    SSHWIFTY_SOCKS5_USER= \
    SSHWIFTY_SOCKS5_PASSWORD= \
    SSHWIFTY_LISTENINTERFACE=0.0.0.0 \
    SSHWIFTY_LISTENPORT=8182 \
    SSHWIFTY_INITIALTIMEOUT=0 \
    SSHWIFTY_READTIMEOUT=0 \
    SSHWIFTY_WRITETIMEOUT=0 \
    SSHWIFTY_HEARTBEATTIMEOUT=0 \
    SSHWIFTY_READDELAY=0 \
    SSHWIFTY_WRITEELAY=0 \
    SSHWIFTY_TLSCERTIFICATEFILE= \
    SSHWIFTY_TLSCERTIFICATEKEYFILE= \
    SSHWIFTY_DOCKER_TLSCERT= \
    SSHWIFTY_DOCKER_TLSCERTKEY= \
    SSHWIFTY_PRESETS= \
    SSHWIFTY_ONLYALLOWPRESETREMOTES=
COPY --from=builder /sshwifty /
COPY . /sshwifty-src
RUN set -ex && \
    adduser -D sshwifty && \
    chmod +x /sshwifty && \
    echo '#!/bin/sh' > /sshwifty.sh && echo '([ -z "$SSHWIFTY_DOCKER_TLSCERT" ] || echo "$SSHWIFTY_DOCKER_TLSCERT" > /tmp/cert); ([ -z "$SSHWIFTY_DOCKER_TLSCERTKEY" ] || echo "$SSHWIFTY_DOCKER_TLSCERTKEY" > /tmp/certkey); if [ -f "/tmp/cert" ] && [ -f "/tmp/certkey" ]; then SSHWIFTY_TLSCERTIFICATEFILE=/tmp/cert SSHWIFTY_TLSCERTIFICATEKEYFILE=/tmp/certkey /sshwifty; else /sshwifty; fi;' >> /sshwifty.sh && chmod +x /sshwifty.sh
USER sshwifty
EXPOSE 8182
ENTRYPOINT [ "/sshwifty.sh" ]
CMD []
```
</code></pre></details>

----


**https://github.com/apache/skywalking/blob/master/docker/agent/Dockerfile.agent**
<details><summary>展开</summary><pre><code>

``` yaml
ARG BASE_IMAGE='adoptopenjdk/openjdk8:alpine'

FROM alpine as build

ENV DIST_NAME=apache-skywalking-apm-bin

ADD "$DIST_NAME.tar.gz" /

RUN mv /$DIST_NAME /skywalking

FROM alpine AS cli

WORKDIR /skywalking/bin

ARG CLI_VERSION=0.6.0

ADD https://archive.apache.org/dist/skywalking/cli/${CLI_VERSION}/skywalking-cli-${CLI_VERSION}-bin.tgz /
RUN tar -zxf /skywalking-cli-${CLI_VERSION}-bin.tgz -C / ; \
    mv /skywalking-cli-${CLI_VERSION}-bin/bin/swctl-${CLI_VERSION}-linux-amd64 /skywalking/bin/swctl

FROM $BASE_IMAGE

LABEL maintainer="kezhenxu94@apache.org"

ENV JAVA_TOOL_OPTIONS=-javaagent:/skywalking/agent/skywalking-agent.jar

WORKDIR /skywalking

COPY --from=build /skywalking/agent /skywalking/agent
COPY --from=cli /skywalking/bin/swctl /skywalking/bin
```
</code></pre></details>

----

**https://github.com/apache/skywalking/blob/master/docker/oap/Dockerfile.oap**
<details><summary>展开</summary><pre><code>

``` yaml
ARG BASE_IMAGE='adoptopenjdk/openjdk11:alpine'

FROM golang:1.14 AS cli

ARG COMMIT_HASH=9f267876493943716434fdaa30047a14c0b5b2d9
ARG CLI_CODE=${COMMIT_HASH}.tar.gz
ARG CLI_CODE_URL=https://github.com/apache/skywalking-cli/archive/${CLI_CODE}

ENV CGO_ENABLED=0
ENV GO111MODULE=on

WORKDIR /cli

ADD ${CLI_CODE_URL} .
RUN tar -xf ${CLI_CODE} --strip 1
RUN rm ${CLI_CODE}

RUN mkdir -p /skywalking/bin/
RUN make linux && mv bin/swctl-latest-linux-amd64 /skywalking/bin/swctl

FROM $BASE_IMAGE

ENV JAVA_OPTS=" -Xms256M " \
    SW_CLUSTER="standalone" \
    SW_STORAGE="h2"

ARG DIST_NAME

COPY "$DIST_NAME.tar.gz" /

RUN set -ex; \
    tar -xzf "$DIST_NAME.tar.gz"; \
    rm -rf "$DIST_NAME.tar.gz"; \
    rm -rf "$DIST_NAME/config/log4j2.xml"; \
    rm -rf "$DIST_NAME/bin"; rm -rf "$DIST_NAME/webapp"; rm -rf "$DIST_NAME/agent"; \
    mkdir "$DIST_NAME/bin"; \
    mv "$DIST_NAME" skywalking;

WORKDIR skywalking

COPY --from=cli /skywalking/bin/swctl ./bin

COPY log4j2.xml config/
COPY docker-entrypoint.sh .
RUN mkdir ext-config; \
    mkdir ext-libs;

EXPOSE 12800 11800 1234

ENTRYPOINT ["sh", "docker-entrypoint.sh"]
```
</code></pre></details>

----

**https://github.com/apache/skywalking/blob/master/docker/ui/Dockerfile.ui**
<details><summary>展开</summary><pre><code>

``` yaml
FROM adoptopenjdk/openjdk11:alpine

ENV DIST_NAME=apache-skywalking-apm-bin \
    JAVA_OPTS=" -Xms256M " \
    SW_OAP_ADDRESS="http://127.0.0.1:12800"

COPY "$DIST_NAME.tar.gz" /

RUN set -ex; \
    apk add bash; \
    tar -xzf "$DIST_NAME.tar.gz"; \
    rm -rf "$DIST_NAME.tar.gz"; \
    rm -rf "$DIST_NAME/config"; \
    rm -rf "$DIST_NAME/bin"; rm -rf "$DIST_NAME/oap-libs"; rm -rf "$DIST_NAME/agent"; \
    mv "$DIST_NAME" skywalking;

WORKDIR skywalking

COPY docker-entrypoint.sh .
COPY logback.xml webapp/

EXPOSE 8080

ENTRYPOINT ["bash", "docker-entrypoint.sh"]
```
</code></pre></details>

----

**https://github.com/88250/solo/blob/master/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
FROM maven:3-jdk-8-alpine as MVN_BUILD

WORKDIR /opt/solo/
ADD . /tmp
RUN cd /tmp && mvn package -DskipTests -Pci -q && mv target/solo/* /opt/solo/ \
&& cp -f /tmp/src/main/resources/docker/* /opt/solo/

FROM openjdk:8-alpine
LABEL maintainer="Liang Ding<845765@qq.com>"

WORKDIR /opt/solo/
COPY --from=MVN_BUILD /opt/solo/ /opt/solo/
RUN apk add --no-cache ca-certificates tzdata

ENV TZ=Asia/Shanghai
ARG git_commit=0
ENV git_commit=$git_commit

EXPOSE 8080

ENTRYPOINT [ "java", "-cp", "lib/*:.", "org.b3log.solo.Server" ]
```
</code></pre></details>

----


**https://github.com/koalaman/shellcheck/blob/master/Dockerfile.multi-arch**
<details><summary>展开</summary><pre><code>

``` yaml
# Alpine image
FROM alpine:latest AS alpine
LABEL maintainer="Vidar Holen <vidar@vidarholen.net>"
ARG tag

# Put the right binary for each architecture into place for the
# multi-architecture docker image.
RUN set -x; \
  arch="$(uname -m)"; \
  echo "arch is $arch"; \
  if [ "${arch}" = 'armv7l' ]; then \
    arch='armv6hf'; \
  fi; \
  url_base='https://github.com/koalaman/shellcheck/releases/download/'; \
  tar_file="${tag}/shellcheck-${tag}.linux.${arch}.tar.xz"; \
  wget "${url_base}${tar_file}" -O - | tar xJf -; \
  mv "shellcheck-${tag}/shellcheck" /bin/; \
  rm -rf "shellcheck-${tag}"; \
  ls -laF /bin/shellcheck

# ShellCheck image
FROM scratch
LABEL maintainer="Vidar Holen <vidar@vidarholen.net>"
WORKDIR /mnt
COPY --from=alpine /bin/shellcheck /bin/
ENTRYPOINT ["/bin/shellcheck"]
```
</code></pre></details>

----


**https://github.com/mgr9525/gokins/blob/master/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
FROM golang:1.16.6-alpine3.14 AS builder
# ENV GOPROXY=https://goproxy.cn,direct
# RUN apk add git build-base && git clone https://gitee.com/gokins/gokins.git /build
RUN apk add git build-base && git clone https://github.com/gokins/gokins.git /build
WORKDIR /build
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o bin/gokins main.go


FROM alpine:latest AS final

ENV GOKINS_WORKPATH=/data/gokins

RUN apk --no-cache add openssl ca-certificates curl git wget \
    && rm -rf /var/cache/apk \
    && mkdir -p /app /data/gokins

COPY --from=builder /build/bin/gokins /app
WORKDIR /app
ENTRYPOINT ["/app/gokins"]
```
</code></pre></details>

----


**https://github.com/hanchuanchuan/goInception/blob/master/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
FROM golang:1.12-alpine as builder
# MAINTAINER hanchuanchuan <chuanchuanhan@gmail.com>

ENV TZ=Asia/Shanghai
ENV LANG="en_US.UTF-8"

RUN apk add --no-cache \
    ca-certificates wget \
    make \
    git \
    gcc \
    musl-dev

RUN wget -q -O /usr/local/bin/dumb-init https://github.com/Yelp/dumb-init/releases/download/v1.2.2/dumb-init_1.2.2_amd64 \
&& wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub \
&& wget -q -O /glibc.apk https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.28-r0/glibc-2.28-r0.apk \
 && chmod +x /usr/local/bin/dumb-init

COPY bin/goInception /goInception
# COPY bin/percona-toolkit.tar.gz /tmp/percona-toolkit.tar.gz
COPY bin/pt-online-schema-change /tmp/pt-online-schema-change
COPY bin/gh-ost /tmp/gh-ost
COPY config/config.toml.default /etc/config.toml

# Executable image
FROM alpine

COPY --from=builder /glibc.apk /glibc.apk
COPY --from=builder /etc/apk/keys/sgerrand.rsa.pub /etc/apk/keys/sgerrand.rsa.pub
COPY --from=builder /goInception /goInception
COPY --from=builder /etc/config.toml /etc/config.toml
COPY --from=builder /usr/local/bin/dumb-init /usr/local/bin/dumb-init

# COPY --from=builder /tmp/percona-toolkit.tar.gz /tmp/percona-toolkit.tar.gz
COPY --from=builder /tmp/pt-online-schema-change /usr/local/bin/pt-online-schema-change
COPY --from=builder /tmp/gh-ost /usr/local/bin/gh-ost

WORKDIR /

EXPOSE 4000

ENV LANG="en_US.UTF-8"
ENV TZ=Asia/Shanghai

# ENV PERCONA_TOOLKIT_VERSION 3.0.4

# && wget -O /tmp/percona-toolkit.tar.gz https://www.percona.com/downloads/percona-toolkit/${PERCONA_TOOLKIT_VERSION}/source/tarball/percona-toolkit-${PERCONA_TOOLKIT_VERSION}.tar.gz \

#RUN set -x \
#  && apk add --no-cache perl perl-dbi perl-dbd-mysql perl-io-socket-ssl perl-term-readkey make tzdata \
#  && tar -xzvf /tmp/percona-toolkit.tar.gz -C /tmp \
#  && cd /tmp/percona-toolkit-${PERCONA_TOOLKIT_VERSION} \
#  && perl Makefile.PL \
#  && make \
#  && make test \
#  && make install \
#  && apk del make \
#  && rm -rf /var/cache/apk/* /tmp/percona-toolkit*


RUN set -x \
  && apk add --no-cache perl perl-dbi perl-dbd-mysql perl-io-socket-ssl perl-term-readkey tzdata /glibc.apk \
  && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone \
  && chmod +x /usr/local/bin/pt-online-schema-change \
  && chmod +x /usr/local/bin/gh-ost

ENTRYPOINT ["/usr/local/bin/dumb-init", "/goInception","--config=/etc/config.toml"]
```
</code></pre></details>

----


**https://github.com/pantsel/konga/blob/master/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
FROM node:12.16-alpine

COPY . /app

WORKDIR /app

RUN apk upgrade --update \
    && apk add bash git ca-certificates \
    && npm install -g bower \
    && npm --unsafe-perm --production install \
    && apk del git \
    && rm -rf /var/cache/apk/* \
        /app/.git \
        /app/screenshots \
        /app/test \
    && adduser -H -S -g "Konga service owner" -D -u 1200 -s /sbin/nologin konga \
    && mkdir /app/kongadata /app/.tmp \
    && chown -R 1200:1200 /app/views /app/kongadata /app/.tmp

EXPOSE 1337

VOLUME /app/kongadata

ENTRYPOINT ["/app/start.sh"]
```
</code></pre></details>

----


**https://github.com/flipped-aurora/gin-vue-admin/tree/master/server**
<details><summary>展开</summary><pre><code>

``` yaml
FROM golang:alpine

WORKDIR /go/src/gin-vue-admin
COPY . .

RUN go generate && go env && go build -o server .

FROM alpine:latest
LABEL MAINTAINER="SliverHorn@sliver_horn@qq.com"

WORKDIR /go/src/gin-vue-admin

COPY --from=0 /go/src/gin-vue-admin ./

EXPOSE 8888

ENTRYPOINT ./server -c config.docker.yaml
```
</code></pre></details>

----

**https://github.com/flipped-aurora/gin-vue-admin/blob/master/web/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
FROM node:12.16.1

WORKDIR /gva_web/
COPY . .

RUN npm install
RUN npm run build

FROM nginx:alpine
LABEL MAINTAINER="SliverHorn@sliver_horn@qq.com"

COPY .docker-compose/nginx/conf.d/my.conf /etc/nginx/conf.d/my.conf
COPY --from=0 /gva_web/dist /usr/share/nginx/html
RUN cat /etc/nginx/nginx.conf
RUN cat /etc/nginx/conf.d/my.conf
RUN ls -al /usr/share/nginx/html
```
</code></pre></details>

----

**https://github.com/hyperledger/blockchain-explorer/blob/main/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
# Copyright Tecnalia Research & Innovation (https://www.tecnalia.com)
# Copyright Tecnalia Blockchain LAB
#
# SPDX-License-Identifier: Apache-2.0

FROM node:13-alpine AS BUILD_IMAGE

# default values pf environment variables
# that are used inside container

ENV DEFAULT_WORKDIR /opt
ENV EXPLORER_APP_PATH $DEFAULT_WORKDIR/explorer

# set default working dir inside container
WORKDIR $EXPLORER_APP_PATH

COPY . .

# install required dependencies by NPM packages:
# current dependencies are: python, make, g++
RUN apk add --no-cache --virtual npm-deps python3 make g++ curl bash && \
    python3 -m ensurepip && \
    rm -r /usr/lib/python*/ensurepip && \
    pip3 install --upgrade pip setuptools && \
    rm -r /root/.cache

# install node-prune (https://github.com/tj/node-prune)
RUN curl -sfL https://install.goreleaser.com/github.com/tj/node-prune.sh | bash -s -- -b /usr/local/bin

# install NPM dependencies
RUN npm install && npm run build && npm prune --production

# build explorer app
RUN cd client && npm install && npm prune --production && yarn build

# remove installed packages to free space
RUN apk del npm-deps
RUN /usr/local/bin/node-prune

RUN rm -rf node_modules/rxjs/src/
RUN rm -rf node_modules/rxjs/bundles/
RUN rm -rf node_modules/rxjs/_esm5/
RUN rm -rf node_modules/rxjs/_esm2015/
RUN rm -rf node_modules/grpc/deps/grpc/third_party/

FROM node:13-alpine

# database configuration
ENV DATABASE_HOST 127.0.0.1
ENV DATABASE_PORT 5432
ENV DATABASE_NAME fabricexplorer
ENV DATABASE_USERNAME hppoc
ENV DATABASE_PASSWD password
ENV EXPLORER_APP_ROOT app

ENV DEFAULT_WORKDIR /opt
ENV EXPLORER_APP_PATH $DEFAULT_WORKDIR/explorer

WORKDIR $EXPLORER_APP_PATH

COPY . .
COPY --from=BUILD_IMAGE $EXPLORER_APP_PATH/dist ./app/
COPY --from=BUILD_IMAGE $EXPLORER_APP_PATH/client/build ./client/build/
COPY --from=BUILD_IMAGE $EXPLORER_APP_PATH/node_modules ./node_modules/

# expose default ports
EXPOSE 8080

# run blockchain explorer main app
CMD npm run app-start && tail -f /dev/null
```
</code></pre></details>

----

**https://github.com/fatedier/frp/blob/dev/dockerfiles/Dockerfile-for-frps**
<details><summary>展开</summary><pre><code>

``` yaml
FROM alpine:3.12.0 AS temp

COPY bin/frps /tmp

RUN chmod -R 777 /tmp/frps


FROM alpine:3.12.0

WORKDIR /app

COPY --from=temp /tmp/frps /usr/bin

ENTRYPOINT ["/usr/bin/frps"]
```
</code></pre></details>

----

**https://github.com/fatedier/frp/blob/dev/dockerfiles/Dockerfile-for-frpc**
<details><summary>展开</summary><pre><code>

``` yaml
  
FROM alpine:3.12.0 AS temp

COPY bin/frpc /tmp

RUN chmod -R 777 /tmp/frpc


FROM alpine:3.12.0

WORKDIR /app

COPY --from=temp /tmp/frpc /usr/bin

ENTRYPOINT ["/usr/bin/frpc"]
```
</code></pre></details>

----

**https://github.com/Kong/docker-kong/blob/master/alpine/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
FROM alpine:3.13

LABEL maintainer="Kong <support@konghq.com>"

ARG ASSET=ce
ENV ASSET $ASSET

ARG EE_PORTS

COPY kong.tar.gz /tmp/kong.tar.gz

ARG KONG_VERSION=2.5.0
ENV KONG_VERSION $KONG_VERSION


ARG KONG_AMD64_SHA="ebe0cf3a3e71d202774ede5083c98e2ae39fae0459d11140f53401a66527e1b7"
ENV KONG_AMD64_SHA $KONG_AMD64_SHA

ARG KONG_ARM64_SHA="131964ce443f2d08dc98191fcc442867f2dee2f741ccee9cc519bb99c765f3cf"
ENV KONG_ARM64_SHA $KONG_ARM64_SHA

RUN set -eux; \
    arch="$(apk --print-arch)"; \
    case "${arch}" in \
      x86_64) arch='amd64'; KONG_SHA256=$KONG_AMD64_SHA ;; \
      aarch64) arch='arm64'; KONG_SHA256=$KONG_ARM64_SHA ;; \
    esac; \
    if [ "$ASSET" = "ce" ] ; then \
      apk add --no-cache --virtual .build-deps curl wget tar ca-certificates \
      && curl -fL "https://download.konghq.com/gateway-${KONG_VERSION%%.*}.x-alpine/kong-$KONG_VERSION.$arch.apk.tar.gz" -o /tmp/kong.tar.gz \
      && echo "$KONG_SHA256  /tmp/kong.tar.gz" | sha256sum -c - \
      && apk del .build-deps; \
    fi; \
    mkdir /kong \
    && tar -C /kong -xzf /tmp/kong.tar.gz \
    && mv /kong/usr/local/* /usr/local \
    && mv /kong/etc/* /etc \
    && rm -rf /kong \
    && apk add --no-cache libstdc++ libgcc openssl pcre perl tzdata libcap zip bash zlib zlib-dev git ca-certificates \
    && adduser -S kong \
    && addgroup -S kong \
    && mkdir -p "/usr/local/kong" \
    && chown -R kong:0 /usr/local/kong \
    && chown kong:0 /usr/local/bin/kong \
    && chmod -R g=u /usr/local/kong \
    && rm -rf /tmp/kong.tar.gz \
    && ln -s /usr/local/openresty/bin/resty /usr/local/bin/resty \
    && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/luajit \
    && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/lua \
    && ln -s /usr/local/openresty/nginx/sbin/nginx /usr/local/bin/nginx \
    && if [ "$ASSET" = "ce" ] ; then \
      kong version; \
    fi

COPY docker-entrypoint.sh /docker-entrypoint.sh

USER kong

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 8000 8443 8001 8444 $EE_PORTS

STOPSIGNAL SIGQUIT

HEALTHCHECK --interval=10s --timeout=10s --retries=10 CMD kong health

CMD ["kong", "docker-start"]
```
</code></pre></details>

----

**https://github.com/Kong/docker-kong/blob/master/centos/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
FROM centos:7
LABEL maintainer="Kong <support@konghq.com>"

ARG ASSET=ce
ENV ASSET $ASSET

ARG EE_PORTS

COPY kong.rpm /tmp/kong.rpm

ARG KONG_VERSION=2.5.0
ENV KONG_VERSION $KONG_VERSION

ARG KONG_SHA256="87b789aed871991b92d264b02ceca3c66246c825c28dd71e73faac7293e43fa2"

RUN set -ex; \
    if [ "$ASSET" = "ce" ] ; then \
      curl -fL https://download.konghq.com/gateway-${KONG_VERSION%%.*}.x-centos-7/Packages/k/kong-$KONG_VERSION.el7.amd64.rpm -o /tmp/kong.rpm \
      && echo "$KONG_SHA256  /tmp/kong.rpm" | sha256sum -c -; \
    fi; \
    yum install -y -q unzip shadow-utils git \
    && yum clean all -q \
    && rm -fr /var/cache/yum/* /tmp/yum_save*.yumtx /root/.pki \
    # Please update the centos install docs if the below line is changed so that
    # end users can properly install Kong along with its required dependencies
    # and that our CI does not diverge from our docs.
    && yum install -y /tmp/kong.rpm \
    && yum clean all \
    && rm /tmp/kong.rpm \
    && chown kong:0 /usr/local/bin/kong \
    && chown -R kong:0 /usr/local/kong \
    && ln -s /usr/local/openresty/bin/resty /usr/local/bin/resty \
    && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/luajit \
    && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/lua \
    && ln -s /usr/local/openresty/nginx/sbin/nginx /usr/local/bin/nginx \
    && if [ "$ASSET" = "ce" ] ; then \
      kong version ; \
    fi

COPY docker-entrypoint.sh /docker-entrypoint.sh

USER kong

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 8000 8443 8001 8444 $EE_PORTS

STOPSIGNAL SIGQUIT

HEALTHCHECK --interval=10s --timeout=10s --retries=10 CMD kong health

CMD ["kong", "docker-start"]
```
</code></pre></details>

----

**https://github.com/Kong/docker-kong/blob/master/ubuntu/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
FROM ubuntu:xenial

ARG ASSET=ce
ENV ASSET $ASSET

ARG EE_PORTS

COPY kong.deb /tmp/kong.deb

ARG KONG_VERSION=2.5.0
ENV KONG_VERSION $KONG_VERSION

RUN set -ex \
    && apt-get update \
    && if [ "$ASSET" = "ce" ] ; then \
      apt-get install -y curl \
      && curl -fL https://download.konghq.com/gateway-${KONG_VERSION%%.*}.x-ubuntu-$(cat /etc/os-release | grep UBUNTU_CODENAME | cut -d = -f 2)/pool/all/k/kong/kong_${KONG_VERSION}_$(dpkg --print-architecture).deb -o /tmp/kong.deb \
      && apt-get purge -y curl; \
    fi; \
    apt-get install -y --no-install-recommends unzip git \
    # Please update the ubuntu install docs if the below line is changed so that
    # end users can properly install Kong along with its required dependencies
    # and that our CI does not diverge from our docs.
    && apt install --yes /tmp/kong.deb \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/kong.deb \
    && chown kong:0 /usr/local/bin/kong \
    && chown -R kong:0 /usr/local/kong \
    && ln -s /usr/local/openresty/bin/resty /usr/local/bin/resty \
    && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/luajit \
    && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/lua \
    && ln -s /usr/local/openresty/nginx/sbin/nginx /usr/local/bin/nginx \
    && if [ "$ASSET" = "ce" ] ; then \
      kong version ; \
    fi

COPY docker-entrypoint.sh /docker-entrypoint.sh

USER kong

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 8000 8443 8001 8444 $EE_PORTS

STOPSIGNAL SIGQUIT

HEALTHCHECK --interval=10s --timeout=10s --retries=10 CMD kong health

CMD ["kong", "docker-start"]
```
</code></pre></details>

----

**https://github.com/tuna/freedns-go/blob/master/Dockerfile**

<details><summary>展开</summary><pre><code>

``` yaml
FROM python:alpine as update_db
WORKDIR /usr/src/app
COPY chinaip .
RUN pip3 install -r requirements.txt
RUN python3 update_db.py

FROM golang:alpine as builder
WORKDIR /go/src/github.com/tuna/freedns-go
COPY go.* ./
RUN go mod download
COPY . .
COPY --from=update_db /usr/src/app/db.go chinaip/
RUN go build -o ./build/freedns-go

FROM alpine
COPY --from=builder /go/src/github.com/tuna/freedns-go/build/freedns-go ./
ENTRYPOINT ["./freedns-go"]
CMD ["-f", "114.114.114.114:53", "-c", "8.8.8.8:53", "-l", "0.0.0.0:53"]
```
</code></pre>
</details>

----
**https://github.com/cptactionhank/docker-atlassian-confluence/blob/master/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
FROM openjdk:8-alpine

# Setup useful environment variables
ENV CONF_HOME     /var/atlassian/confluence
ENV CONF_INSTALL  /opt/atlassian/confluence
ENV CONF_VERSION  7.9.3

ENV JAVA_CACERTS  $JAVA_HOME/jre/lib/security/cacerts
ENV CERTIFICATE   $CONF_HOME/certificate

# Install Atlassian Confluence and helper tools and setup initial home
# directory structure.
RUN set -x \
    && apk --no-cache add curl xmlstarlet bash ttf-dejavu libc6-compat gcompat \
    && mkdir -p                "${CONF_HOME}" \
    && chmod -R 700            "${CONF_HOME}" \
    && chown daemon:daemon     "${CONF_HOME}" \
    && mkdir -p                "${CONF_INSTALL}/conf" \
    && curl -Ls                "https://www.atlassian.com/software/confluence/downloads/binary/atlassian-confluence-${CONF_VERSION}.tar.gz" | tar -xz --directory "${CONF_INSTALL}" --strip-components=1 --no-same-owner \
    && curl -Ls                "https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.44.tar.gz" | tar -xz --directory "${CONF_INSTALL}/confluence/WEB-INF/lib" --strip-components=1 --no-same-owner "mysql-connector-java-5.1.44/mysql-connector-java-5.1.44-bin.jar" \
    && chmod -R 700            "${CONF_INSTALL}/conf" \
    && chmod -R 700            "${CONF_INSTALL}/temp" \
    && chmod -R 700            "${CONF_INSTALL}/logs" \
    && chmod -R 700            "${CONF_INSTALL}/work" \
    && chown -R daemon:daemon  "${CONF_INSTALL}/conf" \
    && chown -R daemon:daemon  "${CONF_INSTALL}/temp" \
    && chown -R daemon:daemon  "${CONF_INSTALL}/logs" \
    && chown -R daemon:daemon  "${CONF_INSTALL}/work" \
    && echo -e                 "\nconfluence.home=$CONF_HOME" >> "${CONF_INSTALL}/confluence/WEB-INF/classes/confluence-init.properties" \
    && xmlstarlet              ed --inplace \
        --delete               "Server/@debug" \
        --delete               "Server/Service/Connector/@debug" \
        --delete               "Server/Service/Connector/@useURIValidationHack" \
        --delete               "Server/Service/Connector/@minProcessors" \
        --delete               "Server/Service/Connector/@maxProcessors" \
        --delete               "Server/Service/Engine/@debug" \
        --delete               "Server/Service/Engine/Host/@debug" \
        --delete               "Server/Service/Engine/Host/Context/@debug" \
                               "${CONF_INSTALL}/conf/server.xml" \
    && touch -d "@0"           "${CONF_INSTALL}/conf/server.xml" \
    && chown daemon:daemon     "${JAVA_CACERTS}"

# Use the default unprivileged account. This could be considered bad practice
# on systems where multiple processes end up being executed by 'daemon' but
# here we only ever run one process anyway.
USER daemon:daemon

# Expose default HTTP connector port.
EXPOSE 8090 8091

# Set volume mount points for installation and home directory. Changes to the
# home directory needs to be persisted as well as parts of the installation
# directory due to eg. logs.
VOLUME ["/var/atlassian/confluence", "/opt/atlassian/confluence/logs"]

# Set the default working directory as the Confluence home directory.
WORKDIR /var/atlassian/confluence

COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]

# Run Atlassian Confluence as a foreground process by default.
CMD ["/opt/atlassian/confluence/bin/start-confluence.sh", "-fg"]
```
</code></pre></details>

----
**https://github.com/jc21/nginx-proxy-manager/blob/develop/docker/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
# This is a Dockerfile intended to be built using `docker buildx`
# for multi-arch support. Building with `docker build` may have unexpected results.

# This file assumes that the frontend has been built using ./scripts/frontend-build

FROM nginxproxymanager/nginx-full:node

ARG TARGETPLATFORM
ARG BUILD_VERSION
ARG BUILD_COMMIT
ARG BUILD_DATE

ENV SUPPRESS_NO_CONFIG_WARNING=1 \
	S6_FIX_ATTRS_HIDDEN=1 \
	S6_BEHAVIOUR_IF_STAGE2_FAILS=1 \
	NODE_ENV=production \
	NPM_BUILD_VERSION="${BUILD_VERSION}" \
	NPM_BUILD_COMMIT="${BUILD_COMMIT}" \
	NPM_BUILD_DATE="${BUILD_DATE}"

RUN echo "fs.file-max = 65535" > /etc/sysctl.conf \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends jq logrotate \
	&& apt-get clean \
	&& rm -rf /var/lib/apt/lists/*

# s6 overlay
COPY scripts/install-s6 /tmp/install-s6
RUN /tmp/install-s6 "${TARGETPLATFORM}" && rm -f /tmp/install-s6

EXPOSE 80 81 443

COPY backend       /app
COPY frontend/dist /app/frontend
COPY global        /app/global

WORKDIR /app
RUN yarn install

# add late to limit cache-busting by modifications
COPY docker/rootfs /

# Remove frontend service not required for prod, dev nginx config as well
RUN rm -rf /etc/services.d/frontend /etc/nginx/conf.d/dev.conf

# Change permission of logrotate config file
RUN chmod 644 /etc/logrotate.d/nginx-proxy-manager

VOLUME [ "/data", "/etc/letsencrypt" ]
ENTRYPOINT [ "/init" ]
HEALTHCHECK --interval=5s --timeout=3s CMD /bin/check-health

LABEL org.label-schema.schema-version="1.0" \
	org.label-schema.license="MIT" \
	org.label-schema.name="nginx-proxy-manager" \
	org.label-schema.description="Docker container for managing Nginx proxy hosts with a simple, powerful interface " \
	org.label-schema.url="https://github.com/jc21/nginx-proxy-manager" \
	org.label-schema.vcs-url="https://github.com/jc21/nginx-proxy-manager.git" \
	org.label-schema.cmd="docker run --rm -ti jc21/nginx-proxy-manager:latest"
```
</code></pre></details>

----
**https://github.com/rtCamp/action-slack-notify/blob/master/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
FROM golang:1.14-alpine3.11@sha256:6578dc0c1bde86ccef90e23da3cdaa77fe9208d23c1bb31d942c8b663a519fa5 AS builder

LABEL "com.github.actions.icon"="bell"
LABEL "com.github.actions.color"="yellow"
LABEL "com.github.actions.name"="Slack Notify"
LABEL "com.github.actions.description"="This action will send notification to Slack"
LABEL "org.opencontainers.image.source"="https://github.com/rtCamp/action-slack-notify"

WORKDIR ${GOPATH}/src/github.com/rtcamp/action-slack-notify
COPY main.go ${GOPATH}/src/github.com/rtcamp/action-slack-notify

ENV CGO_ENABLED 0
ENV GOOS linux

RUN go get -v ./...
RUN go build -a -installsuffix cgo -ldflags '-w  -extldflags "-static"' -o /go/bin/slack-notify .

# alpine:latest at 2020-01-18T01:19:37.187497623Z
FROM alpine@sha256:ab00606a42621fb68f2ed6ad3c88be54397f981a7b70a79db3d1172b11c4367d

COPY --from=builder /go/bin/slack-notify /usr/bin/slack-notify

ENV VAULT_VERSION 1.0.2

RUN apk update \
	&& apk upgrade \
	&& apk add \
	bash \
	jq \
	ca-certificates \
	python \
	py2-pip \
	rsync && \
	pip install shyaml && \
	rm -rf /var/cache/apk/*

# Setup Vault
RUN wget https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_amd64.zip && \
	unzip vault_${VAULT_VERSION}_linux_amd64.zip && \
	rm vault_${VAULT_VERSION}_linux_amd64.zip && \
	mv vault /usr/local/bin/vault

# fix the missing dependency - https://stackoverflow.com/a/35613430
RUN mkdir /lib64 && ln -s /lib/libc.musl-x86_64.so.1 /lib64/ld-linux-x86-64.so.2

COPY *.sh /

RUN chmod +x /*.sh

ENTRYPOINT ["/entrypoint.sh"]
```
</code></pre></details>

----
**https://github.com/tanmng/docker-chevereto/blob/master/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
# Specify the version of PHP we use for our Chevereto
ARG PHP_VERSION=7.4-apache
FROM alpine as downloader

ARG CHEVERETO_VERSION=1.3.0
RUN apk add --no-cache curl && \
    curl -sS -o /tmp/chevereto.zip -L "https://github.com/Chevereto/Chevereto-Free/archive/${CHEVERETO_VERSION}.zip" && \
    mkdir -p /extracted && \
    cd /extracted && \
    unzip /tmp/chevereto.zip  && \
    mv "Chevereto-Free-${CHEVERETO_VERSION}/" Chevereto/
COPY settings.php /extracted/Chevereto/app/settings.php

FROM php:$PHP_VERSION

# Install required packages and configure plugins + mods for Chevereto
RUN apt-get update \
    && apt-get install -y \
        libgd-dev \
        libwebp-dev \
        libzip-dev \
    && bash -c 'if [[ $PHP_VERSION == 7.4.* ]]; then \
      docker-php-ext-configure gd --with-freetype=/usr/include/ --with-jpeg=/usr/include/ --with-webp; \
    else \
      docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/  --with-webp-dir=/usr/include/; \
    fi' && \
    docker-php-ext-install \
        exif \
        gd \
        mysqli \
        pdo \
        pdo_mysql \
        zip && \
    a2enmod rewrite

# Download installer script
COPY --from=downloader --chown=33:33 /extracted/Chevereto /var/www/html

# Expose the image directory as a volume
VOLUME /var/www/html/images

# DB connection environment variables
ENV CHEVERETO_DB_HOST=db CHEVERETO_DB_USERNAME=chevereto CHEVERETO_DB_PASSWORD=chevereto CHEVERETO_DB_NAME=chevereto CHEVERETO_DB_PREFIX=chv_ CHEVERETO_DB_PORT=3306
ARG BUILD_DATE
ARG CHEVERETO_VERSION=1.2.2

# Set all required labels, we set it here to make sure the file is as reusable as possible
LABEL org.label-schema.url="https://github.com/tanmng/docker-chevereto" \
      org.label-schema.name="Chevereto Free" \
      org.label-schema.license="Apache-2.0" \
      org.label-schema.version="${CHEVERETO_VERSION}" \
      org.label-schema.vcs-url="https://github.com/tanmng/docker-chevereto" \
      maintainer="Tan Nguyen <tan.mng90@gmail.com>" \
      build_signature="Chevereto free version ${CHEVERETO_VERSION}; built on ${BUILD_DATE}; Using PHP version ${PHP_VERSION}"
```
</code></pre></details>

----
**https://github.com/tuna/freedns-go/blob/master/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
FROM python:alpine as update_db
WORKDIR /usr/src/app
COPY chinaip .
RUN pip3 install -r requirements.txt
RUN python3 update_db.py

FROM golang:alpine as builder
WORKDIR /go/src/github.com/tuna/freedns-go
COPY go.* ./
RUN go mod download
COPY . .
COPY --from=update_db /usr/src/app/db.go chinaip/
RUN go build -o ./build/freedns-go


FROM alpine
COPY --from=builder /go/src/github.com/tuna/freedns-go/build/freedns-go ./
ENTRYPOINT ["./freedns-go"]
CMD ["-f", "114.114.114.114:53", "-c", "8.8.8.8:53", "-l", "0.0.0.0:53"]
```
</code></pre></details>

----	
**https://github.com/sorenisanerd/gotty/blob/master/Dockerfile**
<details><summary>展开</summary><pre><code>

``` yaml
FROM golang:1.13.1

WORKDIR /gotty
COPY . /gotty
RUN CGO_ENABLED=0 make

FROM alpine:latest

RUN apk update && \
    apk upgrade && \
    apk --no-cache add ca-certificates && \
    apk add bash
WORKDIR /root
COPY --from=0 /gotty/gotty /usr/bin/
CMD ["gotty",  "-w", "bash"]
```
</code></pre></details>
