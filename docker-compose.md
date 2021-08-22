****
<details><summary>展开</summary><pre><code>

```
</code></pre></details>
----

****
<details><summary>展开</summary><pre><code>

```
</code></pre></details>
----

****
<details><summary>展开</summary><pre><code>

```
</code></pre></details>
----

****
<details><summary>展开</summary><pre><code>

```
</code></pre></details>
----

****
<details><summary>展开</summary><pre><code>

```
</code></pre></details>
----

****
<details><summary>展开</summary><pre><code>

```
</code></pre></details>
----

**https://github.com/pantsel/konga/blob/master/docker-compose.yml**
<details><summary>展开</summary><pre><code>
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
