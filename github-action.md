
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


**https://github.com/hanchuanchuan/goInception/blob/master/.github/workflows/go.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: Go
on:
  push:
    branches:
      - master
      - release/*
  pull_request:

jobs:
  build:
    name: Build
    runs-on: ubuntu-16.04
    steps:
      # - name: show mysql info
      #   run: ls -l /etc/my* || true

      - name: Set MySQL Config
        run: |
          sudo service mysql stop
          sudo mkdir -p /etc/mysql/ || true
          echo '[mysqld]'            | sudo tee /etc/mysql/my.cnf
          echo 'server-id=111'         | sudo tee -a /etc/mysql/my.cnf
          echo 'log-bin=on'       | sudo tee -a /etc/mysql/my.cnf
          echo 'binlog-format = row' | sudo tee -a /etc/mysql/my.cnf
          echo 'gtid-mode = ON'      | sudo tee -a /etc/mysql/my.cnf
          echo 'enforce_gtid_consistency = ON' | sudo tee -a /etc/mysql/my.cnf
          echo 'lower_case_table_names = 1' | sudo tee -a /etc/mysql/my.cnf
          echo 'character_set_server = utf8' | sudo tee -a /etc/mysql/my.cnf
          # cat /etc/mysql/my.cnf || true
      - name: Startup MySQL
        run: |
          sudo service mysql start || true
          sudo tail -100 /var/log/mysql/error.log
      - name: Show MySQL Variables
        run: mysql -uroot -p'root' -e "show variables where Variable_name in ('server_id','log_bin','lower_case_table_names','version');"

      # - name: Show MySQL buffer_pool
      #   run: mysql -uroot -p'root' -e "show variables like '%buffer_pool%'"

      - name: Init MySQL Database and User
        run: |
          mysql -uroot -p'root' -e "create database if not exists test DEFAULT CHARACTER SET utf8;create database if not exists test_inc DEFAULT CHARACTER SET utf8;"
          mysql -uroot -p'root' -e "grant all on *.* to test@'127.0.0.1' identified by 'test';FLUSH PRIVILEGES;"
          mysql -uroot -p'root' -e "show databases;show variables like 'explicit_defaults_for_timestamp';"
      # - name: Setup MySQL
      #   uses: samin/mysql-action@v1
      #   with:
      #     host port: 3306 # Optional, default value is 3306. The port of host
      #     container name: mysql
      #     container label: mysql
      #     container port: 3306
      #     character set server: 'utf8mb4'
      #     collation server: 'utf8_general_ci'
      #     lower case table names: 1
      #     log bin: 1
      #     server id: 111
      #     binlog format: 'row'
      #     gtid mode: 1
      #     bind address: '*'
      #     enforce gtid consistency: 1
      #     skip name resolve: 1
      #     mysql version: '5.7'
      #     mysql database: 'test'
      #     mysql root password: 'root'
      #     mysql user: 'test'
      #     mysql password: 'test'

      # - name: docker ps
      #   run: |
      #     date "+%Y-%m-%d %H:%M:%S"
      #     docker ps
      #     # docker logs $(docker ps -q)

      - name: Waiting for MySQL to be ready
        run: |
          sleep 2
          for i in `seq 1 10`;
          do
            nc -z 127.0.0.1 3306 && echo Success && exit 0
            echo -n .
            sleep 2
          done
          echo Failed waiting for MySQL && exit 1
      - name: Set up Go 1.13
        uses: actions/setup-go@v1
        with:
          go-version: 1.13
        id: go

      - name: Check out code into the Go module directory
        uses: actions/checkout@v1

      - name: Install pt-online-schema-change
        run: |
          sudo cp ./bin/pt-online-schema-change  /usr/local/bin/pt-online-schema-change
          sudo chmod +x /usr/local/bin/pt-online-schema-change
          sudo apt-get install libdbi-perl libdbd-mysql-perl
      - name: Install gh-ost
        run: |
          sudo wget -O gh-ost.tar.gz https://github.com/github/gh-ost/releases/download/v1.1.0/gh-ost-binary-linux-20200828140552.tar.gz
          sudo tar -zxvf gh-ost.tar.gz -C /usr/local/bin/
          sudo chmod +x /usr/local/bin/gh-ost
      - name: "Build & Test"
        run: |
          rm -f go.sum
          sudo chmod +x cmd/explaintest/run-tests.sh
          make checklist parserlib gotest
```
</code></pre></details>

----

**https://github.com/lark-parser/lark/blob/master/.github/workflows/tests.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: Tests
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [2.7, 3.5, 3.6, 3.7, 3.8, 3.9, 3.10.0-rc - 3.10, pypy2, pypy3]

    steps:
      - uses: actions/checkout@v2
      - name: Download submodules
        run: |
          git submodule update --init --recursive
          git submodule sync -q
          git submodule update --init
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v2
        with:
          python-version: ${{ matrix.python-version }}
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r test-requirements.txt
      - name: Run tests
        run: |
          python -m tests
```
</code></pre></details>

----


**https://github.com/lark-parser/lark/blob/master/.github/workflows/codecov.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: Compute coverage and push to Codecov
on: [push]
jobs:
  run:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    env:
      OS: ${{ matrix.os }}
      PYTHON: '3.7'
    steps:
    - uses: actions/checkout@master
    - name: Download submodules
      run: |
        git submodule update --init --recursive
        git submodule sync -q
        git submodule update --init
    - name: Setup Python
      uses: actions/setup-python@master
      with:
        python-version: 3.7
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r test-requirements.txt
    - name: Generate coverage report
      run: |
        pip install pytest
        pip install pytest-cov
        pytest --cov=./ --cov-report=xml
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v1
      with:
        token: ${{ secrets.CODECOV_TOKEN }}
        files: ./coverage.xml
        flags: unittests
        env_vars: OS,PYTHON
        name: codecov-umbrella
        fail_ci_if_error: true
        path_to_write_report: ./coverage/codecov_report.txt
        verbose: true
```
</code></pre></details>

----

**https://github.com/alibaba/sealer/blob/main/.github/workflows/release.yml**
<details><summary>展开</summary><pre><code>

``` yaml
on:
  push:
    tags:
      - 'v*' # Push events to matching v*, i.e. v1.0, v20.15.10

name: Release

jobs:
  note:
    name: Pre note
    runs-on: ubuntu-18.04
    timeout-minutes: 5
    outputs:
      stringver: ${{ steps.contentrel.outputs.stringver }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v2
        with:
          ref: ${{ github.ref }}
          path: src/github.com/alibaba/sealer
      - name: stringver
        id: contentrel
        run: |
          RELEASEVER=${{ github.ref }}
          echo "::set-output name=stringver::${RELEASEVER#refs/tags/v}"
        working-directory: src/github.com/alibaba/sealer
      - name: Save release notes
        uses: actions/upload-artifact@v2
        with:
          name: sealer-release-notes
          path: src/github.com/alibaba/sealer/release_note.md
  build:
    name: Build Release Binaries
    runs-on: ${{ matrix.os }}
    needs: [note]
    timeout-minutes: 10

    strategy:
      matrix:
        os: [ubuntu-18.04]

    steps:
      - name: Install Go
        uses: actions/setup-go@v2
        with:
          go-version: '1.14'

      - name: Set env
        shell: bash
        env:
          MOS: ${{ matrix.os }}
        run: |
          releasever=${{ github.ref }}
          releasever="${releasever#refs/tags/}"
          os=linux
          echo "GIT_TAG=${releasever}" >> $GITHUB_ENV
          echo "GOPATH=${{ github.workspace }}" >> $GITHUB_ENV
          echo "OS=${os}" >> $GITHUB_ENV
          echo "${{ github.workspace }}/bin" >> $GITHUB_PATH
      - name: Checkout sealer
        uses: actions/checkout@v2
        with:
          ref: ${{ github.ref }}
          path: src/github.com/alibaba/sealer

      - name: Make linux
        shell: bash
        run: |
            export MULTI_PLATFORM_BUILD=true
            make build
        working-directory: src/github.com/alibaba/sealer

      - name: Save build binaries
        uses: actions/upload-artifact@v2
        with:
          name: sealer-binaries
          path: src/github.com/alibaba/sealer/_output/assets/*.tar.gz*


  release:
    name: Create sealer Release
    runs-on: ubuntu-18.04
    timeout-minutes: 10
    needs: [build, note]

    steps:
      - name: Download builds and release notes
        uses: actions/download-artifact@v2
        with:
          path: builds
      - name: Catalog build assets for upload
        id: catalog
        run: |
          _filenum=1
          for i in "linux-amd64" "linux-arm64"; do
            for f in `ls builds/sealer-binaries/ | grep ${i}`; do
              echo "::set-output name=file${_filenum}::${f}"
              let "_filenum+=1"
            done
          done
      - name: Create Release
        id: create_release
        uses: actions/create-release@v1.1.2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: sealer ${{ needs.note.outputs.stringver }}
          body_path: ./builds/sealer-release-notes/release_note.md
          draft: false
          prerelease: ${{ contains(github.ref, 'beta') || contains(github.ref, 'rc') }}
      - name: Upload Linux sealer linux amd64
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ./builds/sealer-binaries/${{ steps.catalog.outputs.file1 }}
          asset_name: ${{ steps.catalog.outputs.file1 }}
          asset_content_type: application/gzip
      - name: Upload Linux sealer linux amd64 sha256 sum
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ./builds/sealer-binaries/${{ steps.catalog.outputs.file2 }}
          asset_name: ${{ steps.catalog.outputs.file2 }}
          asset_content_type: text/plain
      - name: Upload Linux seautil linux amd64
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ./builds/sealer-binaries/${{ steps.catalog.outputs.file3 }}
          asset_name: ${{ steps.catalog.outputs.file3 }}
          asset_content_type: application/gzip
      - name: Upload Linux seautil linux amd64 sha256 sum
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ./builds/sealer-binaries/${{ steps.catalog.outputs.file4 }}
          asset_name: ${{ steps.catalog.outputs.file4 }}
          asset_content_type: text/plain
      - name: Upload Linux sealer linux arm64
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ./builds/sealer-binaries/${{ steps.catalog.outputs.file5 }}
          asset_name: ${{ steps.catalog.outputs.file5 }}
          asset_content_type: application/gzip
      - name: Upload Linux sealer linux arm64 sha256 sum
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ./builds/sealer-binaries/${{ steps.catalog.outputs.file6 }}
          asset_name: ${{ steps.catalog.outputs.file6 }}
          asset_content_type: text/plain
      - name: Upload Linux seautil linux arm64
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ./builds/sealer-binaries/${{ steps.catalog.outputs.file7 }}
          asset_name: ${{ steps.catalog.outputs.file7 }}
          asset_content_type: application/gzip
      - name: Upload Linux seautil linux arm64 sha256 sum
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ./builds/sealer-binaries/${{ steps.catalog.outputs.file8 }}
          asset_name: ${{ steps.catalog.outputs.file8 }}
          asset_content_type: text/plain
```
</code></pre></details>

----


**https://github.com/alibaba/sealer/blob/main/.github/workflows/go.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: Go

on:
  push:
    branches: "*"
  pull_request:
    branches: "*"
    paths-ignore:
      - 'docs/**'
      - 'vendor/**'
      - '*.md'
      - '*.yml'
jobs:

  build:
    name: ubuntu - Go v1.14
    runs-on: ubuntu-latest

    steps:

      - name: Set up Go
        uses: actions/setup-go@v2
        with:
          go-version: '1.14'
        id: go

      - name: Check out code into the Go module directory
        uses: actions/checkout@v2
        with:
          ref: ${{ github.ref }}
          path: src/github.com/alibaba/sealer
      - name: Check out code lic
        working-directory: src/github.com/alibaba/sealer
        run: |
          wget https://github.com/google/addlicense/releases/download/v1.0.0/addlicense_1.0.0_Linux_x86_64.tar.gz
          tar -zxvf addlicense_1.0.0_Linux_x86_64.tar.gz -C $(go env GOPATH)/bin
          chmod a+x $(go env GOPATH)/bin/addlicense
          rm -rf addlicense_1.0.0_Linux_x86_64.tar.gz
          make license
          modifyCode=$(git status  -s | grep M | wc -l)
          git status  -s
          if [ $modifyCode -eq 0 ] ; then
              echo "Lic check ok"
            else
              echo "Failed git modify files num is $modifyCode. Lic check error,please exec 'make install-addlicense && make license' in your code "
              exit -1
           fi
      - name: Install go ci lint
        run: curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin v1.39.0

      - name: Run Linter
        run: golangci-lint run -v
        working-directory: src/github.com/alibaba/sealer

      - name: Make linux
        shell: bash
        run: |
          export MULTI_PLATFORM_BUILD=true
          make build
        working-directory: src/github.com/alibaba/sealer

      - name: Save build binaries
        uses: actions/upload-artifact@v2
        with:
          name: sealer-binaries
          path: src/github.com/alibaba/sealer/_output/assets/*.tar.gz*
```
</code></pre></details>

----


**https://github.com/alibaba/sealer/blob/main/.github/workflows/e2e-test.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: Sealer-Test

on:
  push:
    branches: "release*"
  issue_comment:
    types:
      - created
jobs:
  build:
    name: test
    runs-on: ubuntu-latest
    if: ${{ (github.event.issue.pull_request && contains(github.event.comment.body, '/test')) || github.event_name == 'push' }}
    env:
      GO111MODULE: on
    steps:
      - name: Github API Request
        id: request
        uses: octokit/request-action@v2.0.2
        with:
          route: ${{ github.event.issue.pull_request.url }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      - name: Get PR informations
        id: pr_data
        run: |
          echo "::set-output name=repo_name::${{ fromJson(steps.request.outputs.data).head.repo.full_name }}"
          echo "::set-output name=repo_clone_url::${{ fromJson(steps.request.outputs.data).head.repo.clone_url }}"
          echo "::set-output name=repo_ssh_url::${{ fromJson(steps.request.outputs.data).head.repo.ssh_url }}"
      - name: Check out code into the Go module directory
        uses: actions/checkout@v2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          repository: ${{fromJson(steps.request.outputs.data).head.repo.full_name}}
          ref: ${{fromJson(steps.request.outputs.data).head.ref}}
          path: src/github.com/alibaba/sealer

      - name: Set up Go 1.13
        uses: actions/setup-go@v1
        with:
          go-version: 1.13
        id: go

      - name: Install sealer and ginkgo
        shell: bash
        run: |
          source hack/build.sh
          export SEALER_DIR=$THIS_PLATFORM_BIN/sealer/linux_amd64
          echo "$SEALER_DIR" >> $GITHUB_PATH
          go get github.com/onsi/ginkgo/ginkgo
          go get github.com/onsi/gomega/...
          GOPATH=`go env GOPATH`
          echo "$GOPATH/bin" >> $GITHUB_PATH
        working-directory: src/github.com/alibaba/sealer

      - name: Run all e2e test
        shell: bash
        env:
          REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
          REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
          REGISTRY_URL: ${{ secrets.REGISTRY_URL }}
          IMAGE_NAME: ${{ secrets.IMAGE_NAME}}
          ACCESSKEYID: ${{ secrets.ACCESSKEYID }}
          ACCESSKEYSECRET: ${{ secrets.ACCESSKEYSECRET }}
          RegionID: ${{ secrets.RegionID }}
        if: ${{ github.event.comment.body == '/test all' || github.event_name == 'push' }}
        run: |
          ginkgo -v test
        working-directory: src/github.com/alibaba/sealer

      - name: Run sealer apply test
        shell: bash
        working-directory: src/github.com/alibaba/sealer
        env:
          REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
          REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
          REGISTRY_URL: ${{ secrets.REGISTRY_URL }}
          IMAGE_NAME: ${{ secrets.IMAGE_NAME}}
          ACCESSKEYID: ${{ secrets.ACCESSKEYID }}
          ACCESSKEYSECRET: ${{ secrets.ACCESSKEYSECRET }}
          RegionID: ${{ secrets.RegionID }}
        if: ${{ github.event.comment.body == '/test apply' }}
        run: |
          ginkgo -v --focus="sealer apply" test
      - name: Run sealer build test
        shell: bash
        working-directory: src/github.com/alibaba/sealer
        env:
          REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
          REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
          REGISTRY_URL: ${{ secrets.REGISTRY_URL }}
          IMAGE_NAME: ${{ secrets.IMAGE_NAME}}
          ACCESSKEYID: ${{ secrets.ACCESSKEYID }}
          ACCESSKEYSECRET: ${{ secrets.ACCESSKEYSECRET }}
          RegionID: ${{ secrets.RegionID }}
        if: ${{ github.event.comment.body == '/test build' }}
        run: |
          ginkgo -v --focus="sealer build" test
      - name: Run sealer run test
        shell: bash
        working-directory: src/github.com/alibaba/sealer
        env:
          REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
          REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
          REGISTRY_URL: ${{ secrets.REGISTRY_URL }}
          IMAGE_NAME: ${{ secrets.IMAGE_NAME}}
          ACCESSKEYID: ${{ secrets.ACCESSKEYID }}
          ACCESSKEYSECRET: ${{ secrets.ACCESSKEYSECRET }}
          RegionID: ${{ secrets.RegionID }}
        if: ${{ github.event.comment.body == '/test run' }}
        run: |
          ginkgo -v --focus="sealer run" test
      - name: Run sealer login test
        shell: bash
        working-directory: src/github.com/alibaba/sealer
        env:
          REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
          REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
          REGISTRY_URL: ${{ secrets.REGISTRY_URL }}
        if: ${{ github.event.comment.body == '/test login' }}
        run: |
          ginkgo -v --focus="sealer login" test
      - name: Run sealer image test
        shell: bash
        working-directory: src/github.com/alibaba/sealer
        env:
          REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
          REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
          REGISTRY_URL: ${{ secrets.REGISTRY_URL }}
          IMAGE_NAME: ${{ secrets.IMAGE_NAME}}
        if: ${{ github.event.comment.body == '/test image' }}
        run: |
          ginkgo -v --focus="sealer image" test
```
</code></pre></details>

----


**https://github.com/alibaba/sealer/blob/main/.github/workflows/auto-invite.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: Invite user to join our group
on:
  issue_comment:
    types:
      - created
jobs:
  issue_comment:
    name: Invite user to join our group
    if: ${{ github.event.comment.body == '/invite' }}
    runs-on: ubuntu-latest
    permissions:
      issues: write
    steps:

      - name: Invite user to join our group
        uses: peter-evans/create-or-update-comment@v1
        with:
          issue-number: ${{ github.event.issue.number }}
          body: |
            It's my pleasure to invite you to join us :
            Sealer dingtalk group : 34619594
            Mail list : sealer@list.alibaba-inc.com
            Developer please add my dingtalk or wechat : fangnux
```
</code></pre></details>

----


**https://github.com/alibaba/sealer/blob/main/.github/workflows/auto-comment.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: Reply dev-quickstart label
on:
  issues:
    types:
      - labeled
jobs:
  add-comment:
    if: github.event.label.name == 'dev-quickstart'
    runs-on: ubuntu-latest
    permissions:
      issues: write
    steps:
      - name: Reply dev-quickstart label
        uses: peter-evans/create-or-update-comment@v1
        with:
          issue-number: ${{ github.event.issue.number }}
          body: |
            If you want to develop this feature, please reply to this issue first and we will assign the task to you.
            [contributing guide](https://github.com/alibaba/sealer/blob/main/CONTRIBUTING.md)
```
</code></pre></details>

----

**https://github.com/flipped-aurora/gin-vue-admin/blob/master/.github/workflows/build_test.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: gin-vue-admin build test

on:
  push:
    branches:
      - "*"
    paths-ignore:
      - "./db/**"
      - "**.md"
  pull_request:
    branches:
      - "*"
    paths-ignore:
      - "./db/**"
      - "**.md"

jobs:
  frontend:
    name: Frontend build
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [12.x]
    steps:
      - name: Check out branch
        uses: actions/checkout@v2

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}

      - name: Build test
        run: |
          npm install
          npm run build
        working-directory: ./web

  backend:
    name: Backend build
    runs-on: ubuntu-latest
    steps:
      - name: Set up Go 1.14
        uses: actions/setup-go@v1
        with:
          go-version: 1.14
        id: go

      - name: Check out branch
        uses: actions/checkout@v2

      - name: Download dependencies
        run: |
          go get -v -t -d ./...
          if [ -f Gopkg.toml ]; then
              curl https://raw.githubusercontent.com/golang/dep/master/install.sh | sh
              dep ensure
          fi
        working-directory: ./server

      - name: Test and Build
        run: |
          go build -v -race
        working-directory: ./server
```
</code></pre></details>

----


**https://github.com/goharbor/harbor/blob/master/.github/workflows/conformance_test.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: CONFORMANCE_TEST
env:
  DOCKER_COMPOSE_VERSION: 1.23.0

on:
  repository_dispatch:
    types:
      - manual-trigger-conformance
  schedule:
    - cron: '0 6 * * *'

jobs:
  CONFORMANCE_TEST:
    env:
      CONFORMANCE_TEST: true
    runs-on:
      #- self-hosted
      - ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: GoogleCloudPlatform/github-actions/setup-gcloud@master
        with:
          version: '285.0.0'
          project_id: ${{ secrets.GCP_PROJECT_ID }}
          service_account_email: ${{ secrets.GCP_SA_EMAIL }}
          service_account_key: ${{ secrets.GCP_SA_KEY }}
          export_default_credentials: true
      - run: gcloud info
      - name: Set up Go 1.16
        uses: actions/setup-go@v1
        with:
          go-version: 1.16.5
        id: go
      - name: setup Docker
        uses: docker-practice/actions-setup-docker@0.0.1
        with:
          docker_version: 18.09
          docker_channel: stable
      - uses: actions/checkout@v2
        with:
          path: src/github.com/goharbor/harbor
      - name: before_install
        run: |
          set -x
          cd src/github.com/goharbor/harbor
          pwd
          env
          df -h
          curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > docker-compose
          chmod +x docker-compose
          sudo mv docker-compose /usr/local/bin
          IP=`hostname -I | awk '{print $1}'`
          echo '{"insecure-registries" : ["'$IP':5000"]}' | sudo tee /etc/docker/daemon.json
          echo "IP=$IP" >> $GITHUB_ENV
          sudo cp ./tests/harbor_ca.crt /usr/local/share/ca-certificates/
          sudo update-ca-certificates
          sudo service docker restart
      - name: install
        run: |
          cd src/github.com/goharbor/harbor
          env
          df -h
          bash ./tests/showtime.sh ./tests/ci/api_common_install.sh $IP DB
      - name: script
        run: |
          echo IP: $IP
          df -h
          cd src/github.com/goharbor/harbor
          bash ./tests/showtime.sh ./tests/ci/conformance_test.sh $IP
          df -h
      - name: upload test result to gs
        run: |
          cd src/github.com/goharbor/harbor
          gsutil cp ./distribution-spec/conformance/report.html gs://harbor-conformance-test/report.html
          gsutil acl ch -u AllUsers:R gs://harbor-conformance-test/report.html
        if: always()
```
</code></pre></details>

----


**https://github.com/goharbor/harbor/blob/master/.github/workflows/codeql-analysis.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: "Code scanning - action"

on:
  push:
  pull_request:
  schedule:
    - cron: '0 16 * * 6'

jobs:
  CodeQL-Build:

    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2
      with:
        # We must fetch at least the immediate parents so that if this is
        # a pull request then we can checkout the head.
        fetch-depth: 2

    # If this run was triggered by a pull request event, then checkout
    # the head of the pull request instead of the merge commit.
    - run: git checkout HEAD^2
      if: ${{ github.event_name == 'pull_request' }}
      
    # Initializes the CodeQL tools for scanning.
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v1
      # Override language selection by uncommenting this and choosing your languages
      # with:
      #   languages: go, javascript, csharp, python, cpp, java

    # Autobuild attempts to build any compiled languages  (C/C++, C#, or Java).
    # If this step fails, then you should remove it and run the build manually (see below)
    #- name: Autobuild
    #  uses: github/codeql-action/autobuild@v1

    # ℹ️ Command-line programs to run using the OS shell.
    # 📚 https://git.io/JvXDl

    # ✏️ If the Autobuild fails above, remove it and uncomment the following three lines
    #    and modify them (or add more) to build your code if your project
    #    uses a compiled language

    #- run: |
    #   make bootstrap
    #   make release

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v1
```
</code></pre></details>

----

**https://github.com/goharbor/harbor/blob/master/.github/workflows/build-package.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: "Build Package Workflow"
env:
  DOCKER_COMPOSE_VERSION: 1.23.0

on:
    push:
        branches:
          - master
          - release-*
        tags:
          - v*
jobs:
  BUILD_PACKAGE:
    env:
        BUILD_PACKAGE: true
    runs-on:
      #- self-hosted
      - ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: google-github-actions/setup-gcloud@master
        with:
          version: '285.0.0'
          project_id: ${{ secrets.GCP_PROJECT_ID }}
          service_account_email: ${{ secrets.GCP_SA_EMAIL }}
          service_account_key: ${{ secrets.GCP_SA_KEY }}
          export_default_credentials: true
      - run: gcloud info
      - name: Set up Go 1.16
        uses: actions/setup-go@v1
        with:
          go-version: 1.16.5
        id: go
      - name: setup Docker
        uses: docker-practice/actions-setup-docker@0.0.1
        with:
          docker_version: 18.09
          docker_channel: stable
      - uses: actions/checkout@v2.1.0
      - uses: jitterbit/get-changed-files@v1
        id: changed-files
        with:
          format: space-delimited
          token: ${{ secrets.GITHUB_TOKEN }}
      - uses: actions/checkout@v2
        with:
          path: src/github.com/goharbor/harbor
      - name: Build Base Image
        if: |
            contains(steps.changed-files.outputs.modified, 'Dockerfile.base') ||
            contains(steps.changed-files.outputs.modified, 'VERSION') ||
            contains(steps.changed-files.outputs.modified, '.buildbaselog')
        run: |
          set -x
          echo "BUILD_BASE=true" >> $GITHUB_ENV
      - name: Build Package
        run: |
          set -x
          env
          df -h
          harbor_target_bucket=""
          target_branch="$(echo ${GITHUB_REF#refs/heads/})"
          harbor_offline_build_bundle=""
          harbor_online_build_bundle=""
          harbor_logs_bucket="harbor-ci-logs"
          harbor_builds_bucket="harbor-builds"
          harbor_releases_bucket="harbor-releases"
          harbor_ci_pipeline_store_bucket="harbor-ci-pipeline-store/latest"
          # the target release version is the version of next release(RC or GA). It needs to be updated on creating new release branch.
          target_release_version=$(cat ./VERSION)
          Harbor_Package_Version=$target_release_version-'build.'$GITHUB_RUN_NUMBER
          if [[ $target_branch == "master" ]]; then
            Harbor_Assets_Version=$Harbor_Package_Version
            harbor_target_bucket=$harbor_builds_bucket
          else
            Harbor_Assets_Version=$target_release_version
            harbor_target_bucket=$harbor_releases_bucket/$target_branch
          fi
          if [[ $target_branch == "release-"* ]]; then
            Harbor_Build_Base_Tag=$target_release_version
          else
            Harbor_Build_Base_Tag=dev
          fi
          build_base_params=" BUILD_BASE=false"
          cd src/github.com/goharbor/harbor
          if [ -z "$BUILD_BASE"  ] || [ "$BUILD_BASE" != "true"  ]; then
            echo "Do not need to build base images!"
          else
            build_base_params=" BUILD_BASE=true PUSHBASEIMAGE=true REGISTRYUSER=\"${{ secrets.DOCKER_HUB_USERNAME }}\" REGISTRYPASSWORD=\"${{ secrets.DOCKER_HUB_PASSWORD }}\""
          fi
          sudo make package_offline GOBUILDTAGS="include_oss include_gcs" BASEIMAGETAG=${Harbor_Build_Base_Tag} VERSIONTAG=${Harbor_Assets_Version} PKGVERSIONTAG=${Harbor_Package_Version} BUILDBIN=true NOTARYFLAG=true CHARTFLAG=true TRIVYFLAG=true HTTPPROXY= ${build_base_params}
          sudo make package_online GOBUILDTAGS="include_oss include_gcs" BASEIMAGETAG=${Harbor_Build_Base_Tag} VERSIONTAG=${Harbor_Assets_Version} PKGVERSIONTAG=${Harbor_Package_Version} BUILDBIN=true NOTARYFLAG=true CHARTFLAG=true TRIVYFLAG=true HTTPPROXY= ${build_base_params}
          harbor_offline_build_bundle=$(basename harbor-offline-installer-*.tgz)
          harbor_online_build_bundle=$(basename harbor-online-installer-*.tgz)
          echo "Package name is: $harbor_offline_build_bundle"
          echo "Package name is: $harbor_online_build_bundle"
          echo -en "${{ secrets.HARBOR_SIGN_KEY }}" |  gpg --import
          gpg -v -ab -u ${{ secrets.HARBOR_SIGN_KEY_ID }} $harbor_offline_build_bundle
          gpg -v -ab -u ${{ secrets.HARBOR_SIGN_KEY_ID }} $harbor_online_build_bundle
          source tests/ci/build_util.sh
          cp ${harbor_offline_build_bundle}                 harbor-offline-installer-latest.tgz
          cp ${harbor_offline_build_bundle}.asc             harbor-offline-installer-latest.tgz.asc
          uploader ${harbor_offline_build_bundle}           $harbor_target_bucket
          uploader ${harbor_offline_build_bundle}.asc       $harbor_target_bucket
          uploader ${harbor_online_build_bundle}            $harbor_target_bucket
          uploader ${harbor_online_build_bundle}.asc        $harbor_target_bucket
          uploader harbor-offline-installer-latest.tgz      $harbor_target_bucket
          uploader harbor-offline-installer-latest.tgz.asc  $harbor_target_bucket
          echo "BUILD_BUNDLE=$harbor_offline_build_bundle" >> $GITHUB_ENV
          publishImage $target_branch $Harbor_Assets_Version "${{ secrets.DOCKER_HUB_USERNAME }}" "${{ secrets.DOCKER_HUB_PASSWORD }}"
      - name: Slack Notification
        uses: sonots/slack-notice-action@v3
        with:
          status: ${{ job.status }}
          title: Build Package - ${{ env.BUILD_BUNDLE }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
        if: always()
```
</code></pre></details>

----

**https://github.com/goharbor/harbor/blob/master/.github/workflows/CI.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: CI
env:
   POSTGRESQL_HOST: localhost
   POSTGRESQL_PORT: 5432
   POSTGRESQL_USR: postgres
   POSTGRESQL_PWD: root123
   POSTGRESQL_DATABASE: registry
   DOCKER_COMPOSE_VERSION: 1.23.0
   HARBOR_ADMIN: admin
   HARBOR_ADMIN_PASSWD: Harbor12345
   CORE_SECRET: tempString
   KEY_PATH: "/data/secret/keys/secretkey"
   REDIS_HOST: localhost
   REG_VERSION: v2.7.1-patch-2819-2553
   UI_BUILDER_VERSION: 1.6.0

on:
  pull_request:
  push:
    paths-ignore:
    - 'docs/**'

jobs:
  UTTEST:
    env:
       UTTEST: true
    runs-on:
      #- self-hosted
      - ubuntu-latest
    timeout-minutes: 100
    steps:
      - name: Set up Go 1.16
        uses: actions/setup-go@v1
        with:
           go-version: 1.16.5
        id: go
      - name: setup Docker
        uses: docker-practice/actions-setup-docker@0.0.1
        with:
          docker_version: 20.04
          docker_channel: stable
      - uses: actions/checkout@v2
        with:
         path: src/github.com/goharbor/harbor
      - name: setup env
        run: |
          cd src/github.com/goharbor/harbor
          pwd
          go env
          echo "GOPATH=$(go env GOPATH):$GITHUB_WORKSPACE" >> $GITHUB_ENV
          echo "$(go env GOPATH)/bin" >> $GITHUB_PATH
          echo "TOKEN_PRIVATE_KEY_PATH=${GITHUB_WORKSPACE}/src/github.com/goharbor/harbor/tests/private_key.pem" >> $GITHUB_ENV
        shell: bash
      - name: before_install
        run: |
          set -x
          cd src/github.com/goharbor/harbor
          pwd
          env
          #sudo apt install -y xvfb
          #xvfb-run ls
          curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > docker-compose
          chmod +x docker-compose
          sudo mv docker-compose /usr/local/bin
          IP=`hostname -I | awk '{print $1}'`
          echo '{"insecure-registries" : ["'$IP':5000"]}' | sudo tee /etc/docker/daemon.json
          echo "IP=$IP" >> $GITHUB_ENV
          sudo cp ./tests/harbor_ca.crt /usr/local/share/ca-certificates/
          sudo update-ca-certificates
          sudo service docker restart
      - name: install
        run: |
          cd src/github.com/goharbor/harbor
          env
          df -h
          bash ./tests/showtime.sh ./tests/ci/ut_install.sh
      - name: script
        run: |
          echo IP: $IP
          df -h
          cd src/github.com/goharbor/harbor
          bash ./tests/showtime.sh ./tests/ci/ut_run.sh $IP
          df -h
      - name: Codecov For BackEnd
        uses: codecov/codecov-action@v1
        with:
          file: ./src/github.com/goharbor/harbor/profile.cov
          flags: unittests

  APITEST_DB:
    env:
      APITEST_DB: true
    runs-on:
      #- self-hosted
      - ubuntu-latest
    timeout-minutes: 100
    steps:
      - name: Set up Go 1.16
        uses: actions/setup-go@v1
        with:
          go-version: 1.16.5
        id: go
      - name: setup Docker
        uses: docker-practice/actions-setup-docker@0.0.1
        with:
          docker_version: 18.09
          docker_channel: stable
      - uses: actions/checkout@v2
        with:
          path: src/github.com/goharbor/harbor
      - name: setup env
        run: |
          cd src/github.com/goharbor/harbor
          pwd
          go env
          echo "GITHUB_TOKEN=${{ secrets.GITHUB_TOKEN }}" >> $GITHUB_ENV
          echo "GOPATH=$(go env GOPATH):$GITHUB_WORKSPACE" >> $GITHUB_ENV
          echo "$(go env GOPATH)/bin" >> $GITHUB_PATH
          echo "TOKEN_PRIVATE_KEY_PATH=${GITHUB_WORKSPACE}/src/github.com/goharbor/harbor/tests/private_key.pem" >> $GITHUB_ENV
          IP=`hostname -I | awk '{print $1}'`
          echo "IP=$IP" >> $GITHUB_ENV
        shell: bash
      - name: before_install
        run: |
          set -x
          cd src/github.com/goharbor/harbor
          pwd
          env
          df -h
          #sudo apt install -y xvfb
          #xvfb-run ls
          curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > docker-compose
          chmod +x docker-compose
          sudo mv docker-compose /usr/local/bin
      - name: install
        run: |
          cd src/github.com/goharbor/harbor
          env
          df -h
          docker system prune -a -f
          bash ./tests/showtime.sh ./tests/ci/api_common_install.sh $IP DB
      - name: script
        run: |
          cd src/github.com/goharbor/harbor
          echo IP: $IP
          df -h
          bash ./tests/showtime.sh ./tests/ci/api_run.sh DB $IP
          df -h

  APITEST_DB_PROXY_CACHE:
    env:
      APITEST_DB: true
    runs-on:
      #- self-hosted
      - ubuntu-latest
    timeout-minutes: 100
    steps:
      - name: Set up Go 1.16
        uses: actions/setup-go@v1
        with:
          go-version: 1.16.5
        id: go
      - name: setup Docker
        uses: docker-practice/actions-setup-docker@0.0.1
        with:
          docker_version: 18.09
          docker_channel: stable
      - uses: actions/checkout@v2
        with:
          path: src/github.com/goharbor/harbor
      - name: setup env
        run: |
          cd src/github.com/goharbor/harbor
          pwd
          go env
          echo "GITHUB_TOKEN=${{ secrets.GITHUB_TOKEN }}" >> $GITHUB_ENV
          echo "GOPATH=$(go env GOPATH):$GITHUB_WORKSPACE" >> $GITHUB_ENV
          echo "$(go env GOPATH)/bin" >> $GITHUB_PATH
          echo "TOKEN_PRIVATE_KEY_PATH=${GITHUB_WORKSPACE}/src/github.com/goharbor/harbor/tests/private_key.pem" >> $GITHUB_ENV
          IP=`hostname -I | awk '{print $1}'`
          echo "IP=$IP" >> $GITHUB_ENV
        shell: bash
      - name: before_install
        run: |
          set -x
          cd src/github.com/goharbor/harbor
          pwd
          env
          df -h
          #sudo apt install -y xvfb
          #xvfb-run ls
          curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > docker-compose
          chmod +x docker-compose
          sudo mv docker-compose /usr/local/bin
      - name: install
        run: |
          cd src/github.com/goharbor/harbor
          env
          df -h
          docker system prune -a -f
          bash ./tests/showtime.sh ./tests/ci/api_common_install.sh $IP DB
      - name: script
        run: |
          cd src/github.com/goharbor/harbor
          echo IP: $IP
          df -h
          bash ./tests/showtime.sh ./tests/ci/api_run.sh PROXY_CACHE $IP
          df -h

  APITEST_LDAP:
    env:
      APITEST_LDAP: true
    runs-on:
      #- self-hosted
      - ubuntu-latest
    timeout-minutes: 100
    steps:
      - name: Set up Go 1.16
        uses: actions/setup-go@v1
        with:
          go-version: 1.16.5
        id: go
      - name: setup Docker
        uses: docker-practice/actions-setup-docker@0.0.1
        with:
          docker_version: 18.09
          docker_channel: stable
      - uses: actions/checkout@v2
        with:
          path: src/github.com/goharbor/harbor
      - name: setup env
        run: |
          cd src/github.com/goharbor/harbor
          pwd
          go env
          echo "GOPATH=$(go env GOPATH):$GITHUB_WORKSPACE" >> $GITHUB_ENV
          echo "$(go env GOPATH)/bin" >> $GITHUB_PATH
          echo "TOKEN_PRIVATE_KEY_PATH=${GITHUB_WORKSPACE}/src/github.com/goharbor/harbor/tests/private_key.pem" >> $GITHUB_ENV
          IP=`hostname -I | awk '{print $1}'`
          echo "IP=$IP" >> $GITHUB_ENV
        shell: bash
      - name: before_install
        run: |
          set -x
          cd src/github.com/goharbor/harbor
          pwd
          env
          df -h
          #sudo apt install -y xvfb
          #xvfb-run ls
          curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > docker-compose
          chmod +x docker-compose
          sudo mv docker-compose /usr/local/bin
      - name: install
        run: |
          cd src/github.com/goharbor/harbor
          env
          df -h
          bash ./tests/showtime.sh ./tests/ci/api_common_install.sh $IP LDAP
      - name: script
        run: |
          echo IP: $IP
          df -h
          cd src/github.com/goharbor/harbor
          bash ./tests/showtime.sh ./tests/ci/api_run.sh LDAP $IP
          df -h

  OFFLINE:
    env:
      OFFLINE: true
    runs-on:
      #- self-hosted
      - ubuntu-latest
    timeout-minutes: 100
    steps:
      - name: Set up Go 1.16
        uses: actions/setup-go@v1
        with:
          go-version: 1.16.5
        id: go
      - name: setup Docker
        uses: docker-practice/actions-setup-docker@0.0.1
        with:
          docker_version: 18.09
          docker_channel: stable
      - uses: actions/checkout@v2
        with:
          path: src/github.com/goharbor/harbor
      - name: setup env
        run: |
          cd src/github.com/goharbor/harbor
          pwd
          docker version
          go env
          echo "GOPATH=$(go env GOPATH):$GITHUB_WORKSPACE" >> $GITHUB_ENV
          echo "$(go env GOPATH)/bin" >> $GITHUB_PATH
          echo "TOKEN_PRIVATE_KEY_PATH=${GITHUB_WORKSPACE}/src/github.com/goharbor/harbor/tests/private_key.pem" >> $GITHUB_ENV
        shell: bash
      - name: before_install
        run: |
          set -x
          cd src/github.com/goharbor/harbor
          pwd
          env
          df -h
          #sudo apt install -y xvfb
          #xvfb-run ls
          curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > docker-compose
          chmod +x docker-compose
          sudo mv docker-compose /usr/local/bin
          IP=`hostname -I | awk '{print $1}'`
          echo '{"insecure-registries" : ["'$IP':5000"]}' | sudo tee /etc/docker/daemon.json
          echo "IP=$IP" >> $GITHUB_ENV
          sudo cp ./tests/harbor_ca.crt /usr/local/share/ca-certificates/
          sudo update-ca-certificates
          sudo service docker restart
      - name: script
        run: |
          echo IP: $IP
          df -h
          cd src/github.com/goharbor/harbor
          bash ./tests/showtime.sh ./tests/ci/distro_installer.sh
          df -h

  UI_UT:
    env:
      UI_UT: true
    runs-on:
      #- self-hosted
      - ubuntu-latest
    timeout-minutes: 100
    steps:
      - uses: actions/setup-node@v1
        with:
          node-version: '16'
      - uses: actions/checkout@v2
        with:
          path: src/github.com/goharbor/harbor
      - name: script
        run: |
          echo IP: $IP
          df -h
          cd src/github.com/goharbor/harbor
          bash ./tests/showtime.sh ./tests/ci/ui_ut_run.sh
          df -h
      - name: Codecov For UI
        uses: codecov/codecov-action@v1
        with:
          file:  ./src/github.com/goharbor/harbor/src/portal/coverage/lcov.info
          flags: unittests
```
</code></pre></details>

----

**https://github.com/sorenisanerd/gotty/blob/master/.github/workflows/pre-release.yaml**
<details><summary>展开</summary><pre><code>

``` yaml
---
name: "pre-release"

on:
  push:
    branches:
      - "master"


jobs:
  pre-release:
    name: "Pre Release"
    runs-on: "ubuntu-latest"

    steps:
    - uses: actions/checkout@v2
      with:
        fetch-depth: 0

    - name: Set up Go
      uses: actions/setup-go@v2
      with:
        go-version: 1.16


    - name: "Build & test"
      run: "make tools release-artifacts"

    - uses: "marvinpinto/action-automatic-releases@latest"
      with:
        repo_token: "${{ secrets.GITHUB_TOKEN }}"
        automatic_release_tag: "latest"
        prerelease: true
        title: "Development Build"
        files: |
          LICENSE
          builds/dist/*
```
</code></pre></details>

----


**https://github.com/sorenisanerd/gotty/blob/master/.github/workflows/main.yaml**
<details><summary>展开</summary><pre><code>

``` yaml
name: Go

on:
  - push

jobs:

  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
      with:
        fetch-depth: 0

    - name: Set up Go
      uses: actions/setup-go@v2
      with:
        go-version: 1.16

    - name: Test
      run: make test

    - name: Build
      run: make tools cross_compile
```
</code></pre></details>

----

**https://github.com/actions/runner/blob/main/.github/workflows/release.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: Runner CD

on:
  workflow_dispatch:
  push:
    paths:
    - releaseVersion
  
jobs:
  check:
    if: startsWith(github.ref, 'refs/heads/releases/') || github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2

    # Make sure ./releaseVersion match ./src/runnerversion 
    # Query GitHub release ensure version is not used 
    - name: Check version
      uses: actions/github-script@0.3.0
      with:
        github-token: ${{secrets.GITHUB_TOKEN}}
        script: |
          const core = require('@actions/core')
          const fs = require('fs');
          const runnerVersion = fs.readFileSync('${{ github.workspace }}/src/runnerversion', 'utf8').replace(/\n$/g, '')
          const releaseVersion = fs.readFileSync('${{ github.workspace }}/releaseVersion', 'utf8').replace(/\n$/g, '')
          if (runnerVersion != releaseVersion) {
            console.log('Request Release Version: ' + releaseVersion + '\nCurrent Runner Version: ' + runnerVersion)
            core.setFailed('Version mismatch! Make sure ./releaseVersion match ./src/runnerVersion')
            return
          }
          try {
            const release = await github.repos.getReleaseByTag({
              owner: '${{ github.event.repository.owner.name }}',
              repo: '${{ github.event.repository.name }}',
              tag: 'v' + runnerVersion
            })
            core.setFailed('Release with same tag already created: ' + release.data.html_url)
          } catch (e) {
            // We are good to create the release if release with same tag doesn't exists
            if (e.status != 404) {
              throw e
            }
          }
  
  build:
    needs: check
    outputs:
      linux-x64-sha: ${{ steps.sha.outputs.linux-x64-sha256 }}
      linux-arm64-sha: ${{ steps.sha.outputs.linux-arm64-sha256 }}
      linux-arm-sha: ${{ steps.sha.outputs.linux-arm-sha256 }}
      win-x64-sha: ${{ steps.sha.outputs.win-x64-sha256 }}
      osx-x64-sha: ${{ steps.sha.outputs.osx-x64-sha256 }}
    strategy:
      matrix:
        runtime: [ linux-x64, linux-arm64, linux-arm, win-x64, osx-x64 ]
        include:
        - runtime: linux-x64
          os: ubuntu-latest
          devScript: ./dev.sh

        - runtime: linux-arm64
          os: ubuntu-latest
          devScript: ./dev.sh

        - runtime: linux-arm
          os: ubuntu-latest
          devScript: ./dev.sh

        - runtime: osx-x64
          os: macOS-latest
          devScript: ./dev.sh

        - runtime: win-x64
          os: windows-latest
          devScript: ./dev

    runs-on: ${{ matrix.os }}
    steps:
    - uses: actions/checkout@v1

    # Build runner layout
    - name: Build & Layout Release
      run: |
        ${{ matrix.devScript }} layout Release ${{ matrix.runtime }}
      working-directory: src

    # Run tests
    - name: L0
      run: |
        ${{ matrix.devScript }} test
      working-directory: src
      if: matrix.runtime != 'linux-arm64' && matrix.runtime != 'linux-arm'

    # Create runner package tar.gz/zip
    - name: Package Release
      if: github.event_name != 'pull_request'
      run: |
        ${{ matrix.devScript }} package Release ${{ matrix.runtime }}
      working-directory: src

    # Upload runner package tar.gz/zip as artifact.
    # Since each package name is unique, so we don't need to put ${{matrix}} info into artifact name
    - name: Publish Artifact
      if: github.event_name != 'pull_request'
      uses: actions/upload-artifact@v1
      with:
        name: runner-packages
        path: _package
    # compute shas and set as job outputs to use in release notes
    - run: brew install coreutils #needed for shasum util
      if: ${{ matrix.os == 'macOS-latest' }}
      name: Install Dependencies for SHA Calculation (osx)
    - run: |
        file=$(ls)
        sha=$(sha256sum $file | awk '{ print $1 }')
        echo "Computed sha256: $sha for $file"
        echo "::set-output name=${{matrix.runtime}}-sha256::$sha"
      shell: bash
      id: sha
      name: Compute SHA256
      working-directory: _package
  release:
    needs: build
    runs-on: ubuntu-latest
    steps:

    - uses: actions/checkout@v2

    # Download runner package tar.gz/zip produced by 'build' job
    - name: Download Artifact
      uses: actions/download-artifact@v1
      with:
        name: runner-packages
        path: ./

    # Create ReleaseNote file
    - name: Create ReleaseNote
      id: releaseNote
      uses: actions/github-script@0.3.0
      with:
        github-token: ${{secrets.GITHUB_TOKEN}}
        script: |
          const core = require('@actions/core')
          const fs = require('fs');
          const runnerVersion = fs.readFileSync('${{ github.workspace }}/src/runnerversion', 'utf8').replace(/\n$/g, '')
          var releaseNote = fs.readFileSync('${{ github.workspace }}/releaseNote.md', 'utf8').replace(/<RUNNER_VERSION>/g, runnerVersion)
          releaseNote = releaseNote.replace(/<WIN_X64_SHA>/g, '${{needs.build.outputs.win-x64-sha}}')
          releaseNote = releaseNote.replace(/<OSX_X64_SHA>/g, '${{needs.build.outputs.osx-x64-sha}}')
          releaseNote = releaseNote.replace(/<LINUX_X64_SHA>/g, '${{needs.build.outputs.linux-x64-sha}}')
          releaseNote = releaseNote.replace(/<LINUX_ARM_SHA>/g, '${{needs.build.outputs.linux-arm-sha}}')
          releaseNote = releaseNote.replace(/<LINUX_ARM64_SHA>/g, '${{needs.build.outputs.linux-arm64-sha}}')
          console.log(releaseNote)
          core.setOutput('version', runnerVersion);
          core.setOutput('note', releaseNote);  
    # Create GitHub release
    - uses: actions/create-release@master
      id: createRelease
      name: Create ${{ steps.releaseNote.outputs.version }} Runner Release
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: "v${{ steps.releaseNote.outputs.version }}"
        release_name: "v${{ steps.releaseNote.outputs.version }}"
        body: |
          ${{ steps.releaseNote.outputs.note }}

    # Upload release assets
    - name: Upload Release Asset (win-x64)
      uses: actions/upload-release-asset@v1.0.1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        upload_url: ${{ steps.createRelease.outputs.upload_url }}
        asset_path: ${{ github.workspace }}/actions-runner-win-x64-${{ steps.releaseNote.outputs.version }}.zip
        asset_name: actions-runner-win-x64-${{ steps.releaseNote.outputs.version }}.zip
        asset_content_type: application/octet-stream

    - name: Upload Release Asset (linux-x64)
      uses: actions/upload-release-asset@v1.0.1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        upload_url: ${{ steps.createRelease.outputs.upload_url }}
        asset_path: ${{ github.workspace }}/actions-runner-linux-x64-${{ steps.releaseNote.outputs.version }}.tar.gz
        asset_name: actions-runner-linux-x64-${{ steps.releaseNote.outputs.version }}.tar.gz
        asset_content_type: application/octet-stream

    - name: Upload Release Asset (osx-x64)
      uses: actions/upload-release-asset@v1.0.1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        upload_url: ${{ steps.createRelease.outputs.upload_url }}
        asset_path: ${{ github.workspace }}/actions-runner-osx-x64-${{ steps.releaseNote.outputs.version }}.tar.gz
        asset_name: actions-runner-osx-x64-${{ steps.releaseNote.outputs.version }}.tar.gz
        asset_content_type: application/octet-stream

    - name: Upload Release Asset (linux-arm)
      uses: actions/upload-release-asset@v1.0.1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        upload_url: ${{ steps.createRelease.outputs.upload_url }}
        asset_path: ${{ github.workspace }}/actions-runner-linux-arm-${{ steps.releaseNote.outputs.version }}.tar.gz
        asset_name: actions-runner-linux-arm-${{ steps.releaseNote.outputs.version }}.tar.gz
        asset_content_type: application/octet-stream

    - name: Upload Release Asset (linux-arm64)
      uses: actions/upload-release-asset@v1.0.1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        upload_url: ${{ steps.createRelease.outputs.upload_url }}
        asset_path: ${{ github.workspace }}/actions-runner-linux-arm64-${{ steps.releaseNote.outputs.version }}.tar.gz
        asset_name: actions-runner-linux-arm64-${{ steps.releaseNote.outputs.version }}.tar.gz
        asset_content_type: application/octet-stream
```
</code></pre></details>

----

**https://github.com/actions/runner/blob/main/.github/workflows/codeql.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: "Code Scanning - Action"

on:
  push:
  pull_request:
  schedule:
    - cron: '0 0 * * 0'

jobs:
  CodeQL-Build:

    strategy:
      fail-fast: false


    # CodeQL runs on ubuntu-latest, windows-latest, and macos-latest
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    # Initializes the CodeQL tools for scanning.
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v1
      # Override language selection by uncommenting this and choosing your languages
      # with:
      #   languages: go, javascript, csharp, python, cpp, java

    - name: Manual build
      run : | 
        ./dev.sh layout Release linux-x64
      working-directory: src

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v1
```
</code></pre></details>

----

**https://github.com/actions/runner/blob/main/.github/workflows/build.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: Runner CI

on:
  workflow_dispatch:
  push:
    branches:
    - main
    - releases/*
    paths-ignore:
    - '**.md'    
  pull_request:
    branches:
    - '*'
    paths-ignore:
    - '**.md'    

jobs:
  build:
    strategy:
      matrix:
        runtime: [ linux-x64, linux-arm64, linux-arm, win-x64, osx-x64 ]
        include:
        - runtime: linux-x64
          os: ubuntu-latest
          devScript: ./dev.sh

        - runtime: linux-arm64
          os: ubuntu-latest
          devScript: ./dev.sh

        - runtime: linux-arm
          os: ubuntu-latest
          devScript: ./dev.sh

        - runtime: osx-x64
          os: macOS-latest
          devScript: ./dev.sh

        - runtime: win-x64
          os: windows-latest
          devScript: ./dev

    runs-on: ${{ matrix.os }}
    steps:
    - uses: actions/checkout@v1

    # Build runner layout
    - name: Build & Layout Release
      run: |
        ${{ matrix.devScript }} layout Release ${{ matrix.runtime }}
      working-directory: src

    # Run tests
    - name: L0
      run: |
        ${{ matrix.devScript }} test
      working-directory: src
      if: matrix.runtime != 'linux-arm64' && matrix.runtime != 'linux-arm'

    # Create runner package tar.gz/zip
    - name: Package Release
      if: github.event_name != 'pull_request'
      run: |
        ${{ matrix.devScript }} package Release
      working-directory: src

    # Upload runner package tar.gz/zip as artifact
    - name: Publish Artifact
      if: github.event_name != 'pull_request'
      uses: actions/upload-artifact@v1
      with:
        name: runner-package-${{ matrix.runtime }}
        path: _package
```
</code></pre></details>

----

****
<details><summary>展开</summary><pre><code>

``` yaml

```
</code></pre></details>

----
