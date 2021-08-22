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


**https://github.com/drone/drone/blob/master/docker/compose/drone-github/docker-compose.yml**
<details><summary>展开</summary><pre><code>

``` yaml
version: "3.8"
services:
    drone:
        image: drone/drone:latest
        ports:
        - "8080:80"
        environment:
        - DRONE_SERVER_HOST=localhost:8080
        - DRONE_SERVER_PROTO=http
        - DRONE_SERVER_PROXY_HOST=${DRONE_SERVER_PROXY_HOST}
        - DRONE_SERVER_PROXY_PROTO=https
        - DRONE_RPC_SECRET=bea26a2221fd8090ea38720fc445eca6
        - DRONE_COOKIE_SECRET=e8206356c843d81e05ab6735e7ebf075
        - DRONE_COOKIE_TIMEOUT=720h
        - DRONE_GITHUB_CLIENT_ID=${DRONE_GITHUB_CLIENT_ID}
        - DRONE_GITHUB_CLIENT_SECRET=${DRONE_GITHUB_CLIENT_SECRET}
        - DRONE_LOGS_DEBUG=true
        - DRONE_CRON_DISABLED=true
        volumes:
        - ./data:/data
    runner:
        image: drone/drone-runner-docker:latest
        environment:
        - DRONE_RPC_HOST=drone
        - DRONE_RPC_PROTO=http
        - DRONE_RPC_SECRET=bea26a2221fd8090ea38720fc445eca6
        - DRONE_TMATE_ENABLED=true
        volumes:
        - /var/run/docker.sock:/var/run/docker.sock
```
</code></pre></details>

----

**https://github.com/hhyo/Archery/blob/master/src/docker-compose/docker-compose.yml**
<details><summary>展开</summary><pre><code>

``` yaml
version: '3'

services:
  redis:
    image: redis:5
    container_name: redis
    restart: always
    command: redis-server --requirepass 123456
    expose:
      - "6379"

  mysql:
    image: mysql:5.7
    container_name: mysql
    restart: always
    ports:
      - "3306:3306"
    volumes:
      - "./mysql/my.cnf:/etc/mysql/my.cnf"
      - "./mysql/datadir:/var/lib/mysql"
    environment:
      MYSQL_DATABASE: archery
      MYSQL_ROOT_PASSWORD: 123456

  inception:
    image: hhyo/inception
    container_name: inception
    restart: always
    expose:
      - "6669"
    volumes:
      - "./inception/inc.cnf:/etc/inc.cnf"

  goinception:
    image: hanchuanchuan/goinception
    container_name: goinception
    restart: always
    expose:
      - "4000"
    volumes:
      - "./inception/config.toml:/etc/config.toml"

  archery:
    image: hhyo/archery:1.8.1
    container_name: archery
    restart: always
    ports:
      - "9123:9123"
    volumes:
      - "./archery/settings.py:/opt/archery/archery/settings.py"
      - "./archery/soar.yaml:/etc/soar.yaml"
      - "./archery/docs.md:/opt/archery/docs/docs.md"
      - "./archery/downloads:/opt/archery/downloads"
      - "./archery/sql/migrations:/opt/archery/sql/migrations"
      - "./archery/logs:/opt/archery/logs"
    entrypoint: "dockerize -wait tcp://mysql:3306 -wait tcp://redis:6379 -timeout 60s /opt/archery/src/docker/startup.sh"
    environment:
      NGINX_PORT: 9123
```
</code></pre></details>

----

**https://github.com/welliamcao/OpsManage/blob/v3/docker/docker-compose.yml**
<details><summary>展开</summary><pre><code>

``` yaml
version: "3"
services:
  db:
    image: mysql:5.6  
    environment:
      - MYSQL_HOST=localhost
      - MYSQL_DATABASE=opsmanage
      - MYSQL_USER=数据库用户名
      - MYSQL_PASSWORD=数据库用户密码
      - MYSQL_ROOT_PASSWORD=数据库root密码
    volumes:
      - /data/apps/mysql:/var/lib/mysql  
    restart: always  
    networks:
      - default
  redis:
     container_name: redis
     image: redis:3.2.8
     command: redis-server 
     ports:
       - "6379:6379"
     volumes:
       - /data/apps/redis:/data/redis
     networks:
       - default  
  rabbitmq:
     container_name: rabbitmq
     image: rabbitmq:management
     ports:
       - "5672:5672"
       - "15672:15672"
     networks:
       - default  

  ops_web:
     image: opsmanage-base:latest
     container_name: ops_web
     environment:
       MYSQL_DATABASE: opsmanage
       MYSQL_USER: "数据库用户名"
       MYSQL_PASSWORD: "数据库用户密码"
     ports:
       - "8000:8000" #vim /mnt/OpsManage/OpsManage/settings.py文件里面的DEBUG设置为DEBUG = True 
     volumes:
       - /mnt/OpsManage:/data/apps/opsmanage
       - /mnt/OpsManage/upload:/data/apps/opsmanage/upload
       - /mnt/OpsManage/logs:/data/apps/opsmanage/logs
     command: bash /data/apps/opsmanage/docker/start.sh  
     links:
       - db
       - redis
       - rabbitmq
     depends_on:
       - db
       - redis
       - rabbitmq
     restart: always
     networks:
       - default  

#  nginx:
#     image: opsmanage-nginx
#     container_name: nginx
#     ports:
#       - "80:80"   
#     volumes:
#       - /mnt/OpsManage/static:/usr/share/nginx/html/static
#     depends_on:
#       - ops_web
#     links:
#       - ops_web:ops_web
#     networks:
#       - default
networks:
  default:
```
</code></pre></details>

----

**https://github.com/apache/skywalking/blob/master/docker/docker-compose.yml**
<details><summary>展开</summary><pre><code>

``` yaml
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

version: '3.5'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:${ES_TAG}
    container_name: elasticsearch
    restart: always
    ports:
      - 9200:9200
    environment:
      discovery.type: single-node
    ulimits:
      memlock:
        soft: -1
        hard: -1
  oap:
    image: skywalking/oap:${TAG}
    container_name: oap
    depends_on:
      - elasticsearch
    links:
      - elasticsearch
    restart: always
    ports:
      - 11800:11800
      - 12800:12800
    environment:
      SW_STORAGE: elasticsearch
      SW_STORAGE_ES_CLUSTER_NODES: elasticsearch:9200
      SW_HEALTH_CHECKER: default
      SW_TELEMETRY: prometheus
    healthcheck:
      test: ["CMD", "./bin/swctl", "ch"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
  ui:
    image: skywalking/ui:${TAG}
    container_name: ui
    depends_on:
      - oap
    links:
      - oap
    restart: always
    ports:
      - 8080:8080
    environment:
      SW_OAP_ADDRESS: http://oap:12800
```
</code></pre></details>

----

**https://github.com/gamegos/cesi/blob/master/docker-compose.yml**
<details><summary>展开</summary><pre><code>

``` yaml
version: "3"
services:
  cesi-backend:
    container_name: cesi-backend
    restart: on-failure
    build:
      context: .
      dockerfile: .docker/Dockerfile.backend
    volumes:
      - "./:/app:ro"
      - "./defaults/cesi.conf.toml:/etc/cesi.conf.toml:ro"
    ports:
      - ${CESI_BACKEND_PORT}:5000
    depends_on:
      - products.example.com
      - analysis.example.com
      - monitoring.example.com

  cesi-frontend:
    container_name: cesi-frontend
    restart: on-failure
    build:
      context: .
      dockerfile: .docker/Dockerfile.frontend
    volumes:
      - "./cesi/ui/:/app:ro"
    ports:
      - "${CESI_UI_PORT}:3000"
    stdin_open: true
    environment:
      - NODE_ENV=development
    depends_on:
      - cesi-backend

  products.example.com:
    image: ef9n/supervisord:${SUPERVISOR_TAG}
    restart: on-failure
    volumes:
      - "./.docker/supervisor/confs/conf.d:/etc/supervisor/conf.d"
      - "./.docker/supervisor/products_supervisord.conf:/etc/supervisor/supervisord.conf"
      - "./.docker/supervisor/bin/:/opt/cesi-dev/bin"
      - "/tmp/cesi-dev/logs/products/:/opt/cesi-dev/logs"
    ports:
      - "9001:9001"

  analysis.example.com:
    image: ef9n/supervisord:${SUPERVISOR_TAG}
    restart: on-failure
    volumes:
      - "./.docker/supervisor/confs/conf.d:/etc/supervisor/conf.d"
      - "./.docker/supervisor/analysis_supervisord.conf:/etc/supervisor/supervisord.conf"
      - "./.docker/supervisor/bin/:/opt/cesi-dev/bin"
      - "/tmp/cesi-dev/logs/analysis/:/opt/cesi-dev/logs"
    ports:
      - "9002:9001"

  monitoring.example.com:
    image: ef9n/supervisord:${SUPERVISOR_TAG}
    restart: on-failure
    volumes:
      - "./.docker/supervisor/confs/conf.d:/etc/supervisor/conf.d"
      - "./.docker/supervisor/monitoring_supervisord.conf:/etc/supervisor/supervisord.conf"
      - "./.docker/supervisor/bin/:/opt/cesi-dev/bin"
      - "/tmp/cesi-dev/logs/monitoring/:/opt/cesi-dev/logs"
    ports:
      - "9003:9001"
```
</code></pre></details>

----

**https://github.com/hanchuanchuan/goInception/blob/master/docker-compose.yml**
<details><summary>展开</summary><pre><code>

``` yaml
version: '2'

services:
  goinception:
    image: hanchuanchuan/goinception
    container_name: goinception
    restart: always
    # 网络模式二选一,使用主机host模式或者映射端口
    network_mode: "host"
    # ports:
    #   - 4000:4000
    volumes:
      # 时区
      # - /etc/localtime:/etc/localtime
      # 配置文件
      - ./config/config.toml:/etc/config.toml
```
</code></pre></details>

----

**https://github.com/pantsel/konga/blob/master/docker-compose.yml**
<details><summary>展开</summary><pre><code>

``` yaml
version: "3"

networks:
 kong-net:
  driver: bridge

services:

  #######################################
  # Postgres: The database used by Kong
  #######################################
  kong-database:
    image: postgres:9.6
    restart: always
    networks:
      - kong-net
    environment:
      POSTGRES_USER: kong
      POSTGRES_DB: kong
      POSTGRES_PASSWORD: kong
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "kong"]
      interval: 5s
      timeout: 5s
      retries: 5

  #######################################
  # Kong database migration
  #######################################
  kong-migration:
    image: kong:1.5
    command: "kong migrations bootstrap"
    networks:
      - kong-net
    restart: on-failure
    environment:
      KONG_PG_HOST: kong-database
      KONG_PG_PASSWORD: kong
    links:
      - kong-database
    depends_on:
      - kong-database

  #######################################
  # Kong: The API Gateway
  #######################################
  kong:
    image: kong:1.5
    restart: always
    networks:
      - kong-net
    environment:
      KONG_PG_HOST: kong-database
      KONG_PG_PASSWORD: kong
      KONG_PROXY_LISTEN: 0.0.0.0:8000
      KONG_PROXY_LISTEN_SSL: 0.0.0.0:8443
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
    depends_on:
      - kong-database
    healthcheck:
      test: ["CMD", "curl", "-f", "http://kong:8001"]
      interval: 5s
      timeout: 2s
      retries: 15
    ports:
      - 8001:8001
      - 8000:8000
```
</code></pre></details>

----


**https://github.com/flipped-aurora/gin-vue-admin/blob/master/docker-compose.yaml**
<details><summary>展开</summary><pre><code>

``` yaml
version: "3"

networks:
  network:
    ipam:
      driver: default
      config:
        - subnet: '177.7.0.0/16'

services:
  web:
    build:
      context: ./web
      dockerfile: ./Dockerfile
    container_name: gva-web
    restart: always
    ports:
      - '8080:8080'
    depends_on:
      - server
    command: [ 'nginx-debug', '-g', 'daemon off;' ]
    networks:
      network:
        ipv4_address: 177.7.0.11

  server:
    build:
      context: ./server
      dockerfile: ./Dockerfile
    container_name: gva-server
    restart: always
    ports:
      - '8888:8888'
    depends_on:
      - mysql
      - redis
    links:
      - mysql
      - redis
    networks:
      network:
        ipv4_address: 177.7.0.12

  mysql:
    image: mysql:8.0.21
    container_name: gva-mysql
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci #设置utf8字符集
    restart: always
    ports:
      - "13306:3306"  # host物理直接映射端口为13306
    environment:
      MYSQL_DATABASE: 'qmPlus' # 初始化启动时要创建的数据库的名称
      MYSQL_ROOT_PASSWORD: 'Aa@6447985' # root管理员用户密码
    networks:
      network:
        ipv4_address: 177.7.0.13

  redis:
    image: redis:6.0.6
    container_name: gva-redis # 容器名
    restart: always
    ports:
      - '16379:6379'
    networks:
      network:
        ipv4_address: 177.7.0.14
```
</code></pre></details>

----
