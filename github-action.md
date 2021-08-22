
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

**https://github.com/jumpserver/jumpserver/blob/master/.github/workflows/release-drafter.yml**
<details><summary>展开</summary><pre><code>

``` yaml
on:
  push:
    # Sequence of patterns matched against refs/tags
    tags:
      - 'v*' # Push events to matching v*, i.e. v1.0, v20.15.10

name: Create Release And Upload assets

jobs:
  create-realese:
    name: Create Release
    runs-on: ubuntu-latest
    outputs:
      upload_url: ${{ steps.create_release.outputs.upload_url }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Get version
        id: get_version
        run: |
          TAG=$(basename ${GITHUB_REF})
          VERSION=${TAG/v/}
          echo "::set-output name=TAG::$TAG"
          echo "::set-output name=VERSION::$VERSION"
      - name: Create Release
        id: create_release
        uses: release-drafter/release-drafter@v5
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          config-name: release-config.yml
          version: ${{ steps.get_version.outputs.TAG }}
          tag: ${{ steps.get_version.outputs.TAG }}

  build-and-release:
    needs: create-realese
    name: Build and Release
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build it and upload
        uses: jumpserver/action-build-upload-assets@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ needs.create-realese.outputs.upload_url }}
```
</code></pre></details>

----

**https://github.com/jumpserver/jumpserver/blob/master/.github/workflows/jms-generic-action-handler.yml**
<details><summary>展开</summary><pre><code>

``` yaml

on: [push, pull_request, release]

name: JumpServer repos generic handler

jobs:
  generic_handler:
    name: Run generic handler
    runs-on: ubuntu-latest
    steps:
      - uses: jumpserver/action-generic-handler@master
        env:
          GITHUB_TOKEN: ${{ secrets.PRIVATE_TOKEN }}
```
</code></pre></details>

----

**https://github.com/ansible/awx/blob/devel/.github/workflows/devel_image.yml**
<details><summary>展开</summary><pre><code>

``` yaml
---
name: Push Development Image
on:
  push:
    branches:
      - devel
jobs:
  push:
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - uses: actions/checkout@v2

      - name: Log in to registry
        run: |
          echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      - name: Pre-pull image to warm build cache
        run: |
          docker pull ghcr.io/${{ github.repository_owner }}/awx_devel:${GITHUB_REF##*/}
      - name: Build image
        run: |
          DEV_DOCKER_TAG_BASE=ghcr.io/${{ github.repository_owner }} COMPOSE_TAG=${GITHUB_REF##*/} make docker-compose-build
      - name: Push image
        run: |
          docker push ghcr.io/${{ github.repository_owner }}/awx_devel:${GITHUB_REF##*/}
```
</code></pre></details>

----


**https://github.com/ansible/awx/blob/devel/.github/workflows/e2e_test.yml**
<details><summary>展开</summary><pre><code>

``` yaml
---
name: E2E Tests
on:
  pull_request_target:
    types: [labeled]
jobs:   
  e2e-test:
    if: contains(github.event.pull_request.labels.*.name, 'qe:e2e')
    runs-on: ubuntu-latest
    timeout-minutes: 40
    permissions:
      packages: write
      contents: read
    strategy:
      matrix:
        job: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24]

    steps:
      - uses: actions/checkout@v2

      - name: Install system deps
        run: sudo apt-get install -y gettext

      - name: Log in to registry
        run: |
          echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      - name: Pre-pull image to warm build cache
        run: |
          docker pull ghcr.io/${{ github.repository_owner }}/awx_devel:${{ github.base_ref }}
      - name: Build UI
        run: |
          DEV_DOCKER_TAG_BASE=ghcr.io/${{ github.repository_owner }} COMPOSE_TAG=${{ github.base_ref }} make ui-devel
      - name: Start AWX
        run: |
          DEV_DOCKER_TAG_BASE=ghcr.io/${{ github.repository_owner }} COMPOSE_TAG=${{ github.base_ref }} make docker-compose &> make-docker-compose-output.log &
      - name: Pull awx_cypress_base image
        run: |
          docker pull quay.io/awx/awx_cypress_base:latest
      - name: Checkout test project
        uses: actions/checkout@v2
        with:
          repository: ${{ github.repository_owner }}/tower-qa
          ssh-key: ${{ secrets.QA_REPO_KEY }}
          path: tower-qa
          ref: devel

      - name: Build cypress
        run: |
          cd ${{ secrets.E2E_PROJECT }}/ui-tests/awx-pf-tests
          docker build -t awx-pf-tests .
      - name: Update default AWX password
        run: |
          while [[ "$(curl -s -o /dev/null -w ''%{http_code}'' -k https://localhost:8043/api/v2/ping/)" != "200" ]]
          do
          echo "Waiting for AWX..."
          sleep 5;
          done
          echo "AWX is up, updating the password..."
          docker exec -i tools_awx_1 sh <<-EOSH
            awx-manage update_password --username=admin --password=password
          EOSH
      - name: Run E2E tests
        env:
          CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}
        run: |
          export COMMIT_INFO_BRANCH=$GITHUB_HEAD_REF
          export COMMIT_INFO_AUTHOR=$GITHUB_ACTOR
          export COMMIT_INFO_SHA=$GITHUB_SHA
          export COMMIT_INFO_REMOTE=$GITHUB_REPOSITORY_OWNER
          cd ${{ secrets.E2E_PROJECT }}/ui-tests/awx-pf-tests
          AWX_IP=$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' tools_awx_1)
          printenv > .env
          echo "Executing tests:"
          docker run \
          --network '_sources_default' \
          --ipc=host \
          --env-file=.env \
          -e CYPRESS_baseUrl="https://$AWX_IP:8043" \
          -e CYPRESS_AWX_E2E_USERNAME=admin \
          -e CYPRESS_AWX_E2E_PASSWORD='password' \
          -e COMMAND="npm run cypress-gha" \
          -v /dev/shm:/dev/shm \
          -v $PWD:/e2e \
          -w /e2e \
          awx-pf-tests run --project .
      - name: Save AWX logs
        uses: actions/upload-artifact@v2
        with:
          name: AWX-logs-${{ matrix.job }}
          path: make-docker-compose-output.log
```
</code></pre></details>

----

**https://github.com/ansible/awx/blob/devel/.github/workflows/ci.yml**
<details><summary>展开</summary><pre><code>

``` yaml
---
name: CI
env:
  BRANCH: ${{ github.base_ref || 'devel' }}
on:
  pull_request:
  push:
    branches: [devel]
jobs:
  api-test:
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - uses: actions/checkout@v2

      - name: Log in to registry
        run: |
          echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      - name: Pre-pull image to warm build cache
        run: |
          docker pull ghcr.io/${{ github.repository_owner }}/awx_devel:${{ env.BRANCH }}
      - name: Build image
        run: |
          DEV_DOCKER_TAG_BASE=ghcr.io/${{ github.repository_owner }} COMPOSE_TAG=${{ env.BRANCH }} make docker-compose-build
      - name: Run API Tests
        run: |
          docker run -u $(id -u) --rm -v ${{ github.workspace}}:/awx_devel/:Z \
            --workdir=/awx_devel ghcr.io/${{ github.repository_owner }}/awx_devel:${{ env.BRANCH }} /start_tests.sh
  api-lint:
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - uses: actions/checkout@v2

      - name: Log in to registry
        run: |
          echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      - name: Pre-pull image to warm build cache
        run: |
          docker pull ghcr.io/${{ github.repository_owner }}/awx_devel:${{ env.BRANCH }}
      - name: Build image
        run: |
          DEV_DOCKER_TAG_BASE=ghcr.io/${{ github.repository_owner }} COMPOSE_TAG=${{ env.BRANCH }} make docker-compose-build
      - name: Run API Linters
        run: |
          docker run -u $(id -u) --rm -v ${{ github.workspace}}:/awx_devel/:Z \
            --workdir=/awx_devel ghcr.io/${{ github.repository_owner }}/awx_devel:${{ env.BRANCH }} /var/lib/awx/venv/awx/bin/tox -e linters
  api-swagger:
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - uses: actions/checkout@v2

      - name: Log in to registry
        run: |
          echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      - name: Pre-pull image to warm build cache
        run: |
          docker pull ghcr.io/${{ github.repository_owner }}/awx_devel:${{ env.BRANCH }} || :
      - name: Build image
        run: |
          DEV_DOCKER_TAG_BASE=ghcr.io/${{ github.repository_owner }} COMPOSE_TAG=${{ env.BRANCH }}  make docker-compose-build
      - name: Generate API Reference
        run: |
          docker run -u $(id -u) --rm -v ${{ github.workspace}}:/awx_devel/:Z \
            --workdir=/awx_devel ghcr.io/${{ github.repository_owner }}/awx_devel:${{ env.BRANCH }} /start_tests.sh swagger
  awx-collection:
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - uses: actions/checkout@v2

      - name: Log in to registry
        run: |
          echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      - name: Pre-pull image to warm build cache
        run: |
          docker pull ghcr.io/${{ github.repository_owner }}/awx_devel:${{ env.BRANCH }}
      - name: Build image
        run: |
          DEV_DOCKER_TAG_BASE=ghcr.io/${{ github.repository_owner }} COMPOSE_TAG=${{ env.BRANCH }}  make docker-compose-build
      - name: Run Collection Tests
        run: |
          docker run -u $(id -u) --rm -v ${{ github.workspace}}:/awx_devel/:Z \
            --workdir=/awx_devel ghcr.io/${{ github.repository_owner }}/awx_devel:${{ env.BRANCH }} /start_tests.sh test_collection_all
  api-schema:
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - uses: actions/checkout@v2

      - name: Log in to registry
        run: |
          echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      - name: Pre-pull image to warm build cache
        run: |
          docker pull ghcr.io/${{ github.repository_owner }}/awx_devel:${{ env.BRANCH }}
      - name: Build image
        run: |
          DEV_DOCKER_TAG_BASE=ghcr.io/${{ github.repository_owner }} COMPOSE_TAG=${{ env.BRANCH }}  make docker-compose-build
      - name: Check API Schema
        run: |
          docker run -u $(id -u) --rm -v ${{ github.workspace}}:/awx_devel/:Z \
            --workdir=/awx_devel ghcr.io/${{ github.repository_owner }}/awx_devel:${{ env.BRANCH }} /start_tests.sh detect-schema-change
  ui-lint:
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - uses: actions/checkout@v2

      - name: Log in to registry
        run: |
          echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      - name: Pre-pull image to warm build cache
        run: |
          docker pull ghcr.io/${{ github.repository_owner }}/awx_devel:${{ env.BRANCH }}
      - name: Build image
        run: |
          DEV_DOCKER_TAG_BASE=ghcr.io/${{ github.repository_owner }} COMPOSE_TAG=${{ env.BRANCH }} make docker-compose-build
      - name: Run UI Linters
        run: |
          docker run -u $(id -u) --rm -v ${{ github.workspace}}:/awx_devel/:Z \
            --workdir=/awx_devel ghcr.io/${{ github.repository_owner }}/awx_devel:${{ env.BRANCH }} make ui-lint
  ui-test:
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - uses: actions/checkout@v2

      - name: Log in to registry
        run: |
          echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      - name: Pre-pull image to warm build cache
        run: |
          docker pull ghcr.io/${{ github.repository_owner }}/awx_devel:${{ env.BRANCH }}
      - name: Build image
        run: |
          DEV_DOCKER_TAG_BASE=ghcr.io/${{ github.repository_owner }} COMPOSE_TAG=${{ env.BRANCH }} make docker-compose-build
      - name: Run UI Tests
        run: |
          docker run -u $(id -u) --rm -v ${{ github.workspace}}:/awx_devel/:Z \
            --workdir=/awx_devel ghcr.io/${{ github.repository_owner }}/awx_devel:${{ env.BRANCH }} make ui-test
```
</code></pre></details>

----

**https://github.com/ctripcorp/apollo/blob/master/.github/workflows/release.yml**
<details><summary>展开</summary><pre><code>

``` yaml
#
# Copyright 2021 Apollo Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# This workflow will build a Java project with Maven
# For more information see: https://help.github.com/actions/language-and-framework-guides/building-and-testing-java-with-maven

name: publish sdks

on:
  workflow_dispatch:
    inputs:
      repository:
        description: 'Maven Repository(snapshots or releases)'
        required: true
        default: 'snapshots'

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Set up Maven Central Repository
      uses: actions/setup-java@v1
      with:
        java-version: 7
        server-id: ${{ github.event.inputs.repository }}
        server-username: MAVEN_USERNAME
        server-password: MAVEN_CENTRAL_TOKEN
        gpg-private-key: ${{ secrets.MAVEN_GPG_PRIVATE_KEY }}
        gpg-passphrase: MAVEN_GPG_PASSPHRASE
    - name: Publish to Apache Maven Central
      run: mvn clean deploy -pl apollo-client,apollo-mockserver,apollo-openapi -am -DskipTests=true "-Dreleases.repo=https://oss.sonatype.org/service/local/staging/deploy/maven2" "-Dsnapshots.repo=https://oss.sonatype.org/content/repositories/snapshots"
      env:
        MAVEN_USERNAME: ${{ secrets.MAVEN_USERNAME }}
        MAVEN_CENTRAL_TOKEN: ${{ secrets.MAVEN_CENTRAL_TOKEN }}
        MAVEN_GPG_PASSPHRASE: ${{ secrets.MAVEN_GPG_PASSPHRASE }}
```
</code></pre></details>

----

**https://github.com/ctripcorp/apollo/blob/master/.github/workflows/cla.yml**
<details><summary>展开</summary><pre><code>

``` yaml
#
# Copyright 2021 Apollo Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
name: "CLA Assistant"
on:
  issue_comment:
    types: [created]
  pull_request_target:
    types: [opened,closed,synchronize]

jobs:
  CLAssistant:
    runs-on: ubuntu-latest
    steps:
      - name: "CLA Assistant"
        if: (github.event.comment.body == 'recheck' || startsWith(github.event.comment.body, 'I have read the CLA Document and I hereby sign the CLA')) || github.event_name == 'pull_request_target'
        # Beta Release
        uses: cla-assistant/github-action@v2.1.2-beta
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          # the below token should have repo scope and must be manually added by you in the repository's secret
          PERSONAL_ACCESS_TOKEN : ${{ secrets.PERSONAL_ACCESS_TOKEN_FOR_CLA_ASSISTANT }}
        with:
          path-to-signatures: 'signatures/version1/cla.json'
          path-to-document: 'https://github.com/ctripcorp/apollo-community/blob/master/CLA.md' # e.g. a CLA or a DCO document
          # branch should not be protected
          branch: 'master'
          allowlist: dependabot,bot*
          remote-repository-name: apollo-community

         #below are the optional inputs - If the optional inputs are not given, then default values will be taken
          #remote-organization-name: enter the remote organization name where the signatures should be stored (Default is storing the signatures in the same repository)
          #remote-repository-name:  enter the  remote repository name where the signatures should be stored (Default is storing the signatures in the same repository)
          #create-file-commit-message: 'For example: Creating file for storing CLA Signatures'
          #signed-commit-message: 'For example: $contributorName has signed the CLA in #$pullRequestNo'
          #custom-notsigned-prcomment: 'pull request comment with Introductory message to ask new contributors to sign'
          #custom-pr-sign-comment: 'The signature to be committed in order to sign the CLA'
          #custom-allsigned-prcomment: 'pull request comment when all contributors has signed, defaults to **CLA Assistant Lite bot** All Contributors have signed the CLA.'
          #lock-pullrequest-aftermerge: false - if you don't want this bot to automatically lock the pull request after merging (default - true)
          #use-dco-flag: true - If you are using DCO instead of CLA
```
</code></pre></details>

----

**https://github.com/ctripcorp/apollo/blob/master/.github/workflows/build.yml**
<details><summary>展开</summary><pre><code>

``` yaml
#
# Copyright 2021 Apollo Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# This workflow will build a Java project with Maven
# For more information see: https://help.github.com/actions/language-and-framework-guides/building-and-testing-java-with-maven

name: build

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        jdk: [7, 8, 11]
    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK
      uses: actions/setup-java@v1
      with:
        java-version: ${{ matrix.jdk }}
    - name: Cache Maven packages
      uses: actions/cache@v1
      with:
        path: ~/.m2
        key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
        restore-keys: ${{ runner.os }}-m2
    - name: Build SDK with JDK 7
      if: matrix.jdk == '7'
      run: mvn clean compile -pl apollo-client,apollo-mockserver,apollo-openapi -am -Dmaven.gitcommitid.skip=true
    - name: JDK 8
      if: matrix.jdk == '8'
      run: mvn -B clean package -P travis jacoco:report -Dmaven.gitcommitid.skip=true
    - name: JDK 11 
      if: matrix.jdk == '11'
      run: mvn clean compile -Dmaven.gitcommitid.skip=true
    - name: Upload coverage to Codecov
      if: matrix.jdk == '8'
      uses: codecov/codecov-action@v1
      with:
        file: ${{ github.workspace }}/apollo-*/target/site/jacoco/jacoco.xml
```
</code></pre></details>

----

**https://github.com/88250/solo/blob/master/.github/workflows/dockerimage.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: Docker Image CI
on: 
  push:
    branches:
      - master
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@master
    - name: Build the Docker image
      run: |
        docker login --username=${{ secrets.DOCKER_HUB_USER }} --password=${{ secrets.DOCKER_HUB_PWD }}
        docker build --build-arg git_commit=$(git rev-parse --short HEAD) -t b3log/solo:latest .
        docker push b3log/solo
```
</code></pre></details>

----

**https://github.com/git/git/blob/master/.github/workflows/main.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: CI/PR

on: [push, pull_request]

env:
  DEVELOPER: 1

jobs:
  ci-config:
    runs-on: ubuntu-latest
    outputs:
      enabled: ${{ steps.check-ref.outputs.enabled }}${{ steps.skip-if-redundant.outputs.enabled }}
    steps:
      - name: try to clone ci-config branch
        run: |
          git -c protocol.version=2 clone \
            --no-tags \
            --single-branch \
            -b ci-config \
            --depth 1 \
            --no-checkout \
            --filter=blob:none \
            https://github.com/${{ github.repository }} \
            config-repo &&
          cd config-repo &&
          git checkout HEAD -- ci/config || : ignore
      - id: check-ref
        name: check whether CI is enabled for ref
        run: |
          enabled=yes
          if test -x config-repo/ci/config/allow-ref &&
             ! config-repo/ci/config/allow-ref '${{ github.ref }}'
          then
            enabled=no
          fi
          echo "::set-output name=enabled::$enabled"
      - name: skip if the commit or tree was already tested
        id: skip-if-redundant
        uses: actions/github-script@v3
        if: steps.check-ref.outputs.enabled == 'yes'
        with:
          github-token: ${{secrets.GITHUB_TOKEN}}
          script: |
            try {
              // Figure out workflow ID, commit and tree
              const { data: run } = await github.actions.getWorkflowRun({
                owner: context.repo.owner,
                repo: context.repo.repo,
                run_id: context.runId,
              });
              const workflow_id = run.workflow_id;
              const head_sha = run.head_sha;
              const tree_id = run.head_commit.tree_id;
              // See whether there is a successful run for that commit or tree
              const { data: runs } = await github.actions.listWorkflowRuns({
                owner: context.repo.owner,
                repo: context.repo.repo,
                per_page: 500,
                status: 'success',
                workflow_id,
              });
              for (const run of runs.workflow_runs) {
                if (head_sha === run.head_sha) {
                  core.warning(`Successful run for the commit ${head_sha}: ${run.html_url}`);
                  core.setOutput('enabled', ' but skip');
                  break;
                }
                if (run.head_commit && tree_id === run.head_commit.tree_id) {
                  core.warning(`Successful run for the tree ${tree_id}: ${run.html_url}`);
                  core.setOutput('enabled', ' but skip');
                  break;
                }
              }
            } catch (e) {
              core.warning(e);
            }
  windows-build:
    needs: ci-config
    if: needs.ci-config.outputs.enabled == 'yes'
    runs-on: windows-latest
    steps:
    - uses: actions/checkout@v2
    - uses: git-for-windows/setup-git-for-windows-sdk@v1
    - name: build
      shell: bash
      env:
        HOME: ${{runner.workspace}}
        NO_PERL: 1
      run: ci/make-test-artifacts.sh artifacts
    - name: zip up tracked files
      run: git archive -o artifacts/tracked.tar.gz HEAD
    - name: upload tracked files and build artifacts
      uses: actions/upload-artifact@v2
      with:
        name: windows-artifacts
        path: artifacts
  windows-test:
    runs-on: windows-latest
    needs: [windows-build]
    strategy:
      fail-fast: false
      matrix:
        nr: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    steps:
    - name: download tracked files and build artifacts
      uses: actions/download-artifact@v2
      with:
        name: windows-artifacts
        path: ${{github.workspace}}
    - name: extract tracked files and build artifacts
      shell: bash
      run: tar xf artifacts.tar.gz && tar xf tracked.tar.gz
    - uses: git-for-windows/setup-git-for-windows-sdk@v1
    - name: test
      shell: bash
      run: ci/run-test-slice.sh ${{matrix.nr}} 10
    - name: ci/print-test-failures.sh
      if: failure()
      shell: bash
      run: ci/print-test-failures.sh
    - name: Upload failed tests' directories
      if: failure() && env.FAILED_TEST_ARTIFACTS != ''
      uses: actions/upload-artifact@v2
      with:
        name: failed-tests-windows
        path: ${{env.FAILED_TEST_ARTIFACTS}}
  vs-build:
    needs: ci-config
    if: needs.ci-config.outputs.enabled == 'yes'
    env:
      NO_PERL: 1
      GIT_CONFIG_PARAMETERS: "'user.name=CI' 'user.email=ci@git'"
    runs-on: windows-latest
    steps:
    - uses: actions/checkout@v2
    - uses: git-for-windows/setup-git-for-windows-sdk@v1
    - name: initialize vcpkg
      uses: actions/checkout@v2
      with:
        repository: 'microsoft/vcpkg'
        path: 'compat/vcbuild/vcpkg'
    - name: download vcpkg artifacts
      shell: powershell
      run: |
        $urlbase = "https://dev.azure.com/git/git/_apis/build/builds"
        $id = ((Invoke-WebRequest -UseBasicParsing "${urlbase}?definitions=9&statusFilter=completed&resultFilter=succeeded&`$top=1").content | ConvertFrom-JSON).value[0].id
        $downloadUrl = ((Invoke-WebRequest -UseBasicParsing "${urlbase}/$id/artifacts").content | ConvertFrom-JSON).value[0].resource.downloadUrl
        (New-Object Net.WebClient).DownloadFile($downloadUrl, "compat.zip")
        Expand-Archive compat.zip -DestinationPath . -Force
        Remove-Item compat.zip
    - name: add msbuild to PATH
      uses: microsoft/setup-msbuild@v1
    - name: copy dlls to root
      shell: cmd
      run: compat\vcbuild\vcpkg_copy_dlls.bat release
    - name: generate Visual Studio solution
      shell: bash
      run: |
        cmake `pwd`/contrib/buildsystems/ -DCMAKE_PREFIX_PATH=`pwd`/compat/vcbuild/vcpkg/installed/x64-windows \
        -DNO_GETTEXT=YesPlease -DPERL_TESTS=OFF -DPYTHON_TESTS=OFF -DCURL_NO_CURL_CMAKE=ON
    - name: MSBuild
      run: msbuild git.sln -property:Configuration=Release -property:Platform=x64 -maxCpuCount:4 -property:PlatformToolset=v142
    - name: bundle artifact tar
      shell: bash
      env:
        MSVC: 1
        VCPKG_ROOT: ${{github.workspace}}\compat\vcbuild\vcpkg
      run: |
        mkdir -p artifacts &&
        eval "$(make -n artifacts-tar INCLUDE_DLLS_IN_ARTIFACTS=YesPlease ARTIFACTS_DIRECTORY=artifacts NO_GETTEXT=YesPlease 2>&1 | grep ^tar)"
    - name: zip up tracked files
      run: git archive -o artifacts/tracked.tar.gz HEAD
    - name: upload tracked files and build artifacts
      uses: actions/upload-artifact@v2
      with:
        name: vs-artifacts
        path: artifacts
  vs-test:
    runs-on: windows-latest
    needs: vs-build
    strategy:
      fail-fast: false
      matrix:
        nr: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    steps:
    - uses: git-for-windows/setup-git-for-windows-sdk@v1
    - name: download tracked files and build artifacts
      uses: actions/download-artifact@v2
      with:
        name: vs-artifacts
        path: ${{github.workspace}}
    - name: extract tracked files and build artifacts
      shell: bash
      run: tar xf artifacts.tar.gz && tar xf tracked.tar.gz
    - name: test
      shell: bash
      env:
        NO_SVN_TESTS: 1
        GIT_TEST_SKIP_REBASE_P: 1
      run: ci/run-test-slice.sh ${{matrix.nr}} 10
    - name: ci/print-test-failures.sh
      if: failure()
      shell: bash
      run: ci/print-test-failures.sh
    - name: Upload failed tests' directories
      if: failure() && env.FAILED_TEST_ARTIFACTS != ''
      uses: actions/upload-artifact@v2
      with:
        name: failed-tests-windows
        path: ${{env.FAILED_TEST_ARTIFACTS}}
  regular:
    needs: ci-config
    if: needs.ci-config.outputs.enabled == 'yes'
    strategy:
      fail-fast: false
      matrix:
        vector:
          - jobname: linux-clang
            cc: clang
            pool: ubuntu-latest
          - jobname: linux-gcc
            cc: gcc
            pool: ubuntu-latest
          - jobname: osx-clang
            cc: clang
            pool: macos-latest
          - jobname: osx-gcc
            cc: gcc
            pool: macos-latest
          - jobname: linux-gcc-default
            cc: gcc
            pool: ubuntu-latest
    env:
      CC: ${{matrix.vector.cc}}
      jobname: ${{matrix.vector.jobname}}
    runs-on: ${{matrix.vector.pool}}
    steps:
    - uses: actions/checkout@v2
    - run: ci/install-dependencies.sh
    - run: ci/run-build-and-tests.sh
    - run: ci/print-test-failures.sh
      if: failure()
    - name: Upload failed tests' directories
      if: failure() && env.FAILED_TEST_ARTIFACTS != ''
      uses: actions/upload-artifact@v2
      with:
        name: failed-tests-${{matrix.vector.jobname}}
        path: ${{env.FAILED_TEST_ARTIFACTS}}
  dockerized:
    needs: ci-config
    if: needs.ci-config.outputs.enabled == 'yes'
    strategy:
      fail-fast: false
      matrix:
        vector:
        - jobname: linux-musl
          image: alpine
        - jobname: Linux32
          image: daald/ubuntu32:xenial
    env:
      jobname: ${{matrix.vector.jobname}}
    runs-on: ubuntu-latest
    container: ${{matrix.vector.image}}
    steps:
    - uses: actions/checkout@v1
    - run: ci/install-docker-dependencies.sh
    - run: ci/run-build-and-tests.sh
    - run: ci/print-test-failures.sh
      if: failure()
    - name: Upload failed tests' directories
      if: failure() && env.FAILED_TEST_ARTIFACTS != ''
      uses: actions/upload-artifact@v2
      with:
        name: failed-tests-${{matrix.vector.jobname}}
        path: ${{env.FAILED_TEST_ARTIFACTS}}
  static-analysis:
    needs: ci-config
    if: needs.ci-config.outputs.enabled == 'yes'
    env:
      jobname: StaticAnalysis
    runs-on: ubuntu-18.04
    steps:
    - uses: actions/checkout@v2
    - run: ci/install-dependencies.sh
    - run: ci/run-static-analysis.sh
  sparse:
    needs: ci-config
    if: needs.ci-config.outputs.enabled == 'yes'
    env:
      jobname: sparse
    runs-on: ubuntu-20.04
    steps:
    - name: Download a current `sparse` package
      # Ubuntu's `sparse` version is too old for us
      uses: git-for-windows/get-azure-pipelines-artifact@v0
      with:
        repository: git/git
        definitionId: 10
        artifact: sparse-20.04
    - name: Install the current `sparse` package
      run: sudo dpkg -i sparse-20.04/sparse_*.deb
    - uses: actions/checkout@v2
    - name: Install other dependencies
      run: ci/install-dependencies.sh
    - run: make sparse
  documentation:
    needs: ci-config
    if: needs.ci-config.outputs.enabled == 'yes'
    env:
      jobname: Documentation
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - run: ci/install-dependencies.sh
    - run: ci/test-documentation.sh
```
</code></pre></details>

----

**https://github.com/git/git/blob/master/.github/workflows/check-whitespace.yml**
<details><summary>展开</summary><pre><code>

``` yaml

name: check-whitespace

# Get the repo with the commits(+1) in the series.
# Process `git log --check` output to extract just the check errors.
# Add a comment to the pull request with the check errors.

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  check-whitespace:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
      with:
        fetch-depth: 0

    - name: git log --check
      id: check_out
      run: |
        log=
        commit=
        while read dash etc
        do
          case "${dash}" in
          "---")
            commit="${etc}"
            ;;
          "")
            ;;
          *)
            if test -n "${commit}"
            then
              log="${log}\n${commit}"
              echo ""
              echo "--- ${commit}"
            fi
            commit=
            log="${log}\n${dash} ${etc}"
            echo "${dash} ${etc}"
            ;;
          esac
        done <<< $(git log --check --pretty=format:"---% h% s" ${{github.event.pull_request.base.sha}}..)
        if test -n "${log}"
        then
          exit 2
        fi
```
</code></pre></details>

----


**https://github.com/koalaman/shellcheck/blob/master/.github/workflows/build.yml**
<details><summary>展开</summary><pre><code>

``` yaml
name: Build ShellCheck

# Run this workflow every time a new commit pushed to your repository
on: push

jobs:
  package_source:
    name: Package Source Code
    runs-on: ubuntu-latest
    steps:
      - name: Install Dependencies
        run: |
          sudo apt-get update
          sudo apt-mark manual ghc # Don't bother installing ghc just to tar up source
          sudo apt-get install cabal-install
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Package Source
        run: |
          mkdir source
          cabal sdist
          mv dist-newstyle/sdist/*.tar.gz source/source.tar.gz
      - name: Deduce tags
        run: |
          exec > source/tags
          echo "latest"
          if tag=$(git describe --exact-match --tags)
          then
            echo "stable"
            echo "$tag"
          fi
      - name: Upload artifact
        uses: actions/upload-artifact@v2
        with:
          name: source
          path: source/

  build_source:
    name: Build Source Code
    needs: package_source
    strategy:
      matrix:
        build: [linux.x86_64, linux.aarch64, linux.armv6hf, darwin.x86_64, windows.x86_64]
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Download artifacts
        uses: actions/download-artifact@v2

      - name: Build source
        run: |
          mkdir -p bin
          mkdir -p bin/${{matrix.build}}
          ( cd bin && ../build/run_builder ../source/source.tar.gz ../build/${{matrix.build}} )
      - name: Upload artifact
        uses: actions/upload-artifact@v2
        with:
          name: bin
          path: bin/

  package_binary:
    name: Package Binaries
    needs: build_source
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Download artifacts
        uses: actions/download-artifact@v2

      - name: Work around GitHub permissions bug
        run: chmod +x bin/*/shellcheck*

      - name: Package binaries
        run: |
          export TAGS="$(cat source/tags)"
          mkdir -p deploy
          cp -r bin/* deploy
          cd deploy
          ../.prepare_deploy
          rm -rf */ README* LICENSE*
      - name: Upload artifact
        uses: actions/upload-artifact@v2
        with:
          name: deploy
          path: deploy/

  deploy:
    name: Deploy binaries
    needs: package_binary
    runs-on: ubuntu-latest
    environment: Deploy
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Download artifacts
        uses: actions/download-artifact@v2

      - name: Upload to GitHub
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          export TAGS="$(cat source/tags)"
          ./.github_deploy
      - name: Waiting for GitHub to replicate uploaded releases
        run: |
          sleep 300
      - name: Upload to Docker Hub
        env:
          DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
          DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
          DOCKER_EMAIL: ${{ secrets.DOCKER_EMAIL }}
          DOCKER_BASE: ${{ secrets.DOCKER_USERNAME }}/shellcheck
        run: |
          export TAGS="$(cat source/tags)"
          ( source ./.multi_arch_docker && set -eux && multi_arch_docker::main )
```
</code></pre></details>

----


**https://github.com/jenkinsci/gitlab-plugin/blob/master/.github/workflows/codeql-analysis.yml**
<details><summary>展开</summary><pre><code>

``` yaml
# For most projects, this workflow file will not need changing; you simply need
# to commit it to your repository.
#
# You may wish to alter this file to override the set of languages analyzed,
# or to provide custom queries or build logic.
name: "CodeQL"

on:
  push:
    branches: [master]
  pull_request:
    # The branches below must be a subset of the branches above
    branches: [master]
  schedule:
    - cron: '0 22 * * 6'

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest

    strategy:
      fail-fast: false
      matrix:
        # Override automatic language detection by changing the below list
        # Supported options are ['csharp', 'cpp', 'go', 'java', 'javascript', 'python']
        language: ['java']
        # Learn more...
        # https://docs.github.com/en/github/finding-security-vulnerabilities-and-errors-in-your-code/configuring-code-scanning#overriding-automatic-language-detection

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    # Initializes the CodeQL tools for scanning.
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v1
      with:
        languages: ${{ matrix.language }}
        # If you wish to specify custom queries, you can do so here or in a config file.
        # By default, queries listed here will override any specified in a config file.
        # Prefix the list here with "+" to use these queries and those in the config file.
        # queries: ./path/to/local/query, your-org/your-repo/queries@main

    # Autobuild attempts to build any compiled languages  (C/C++, C#, or Java).
    # If this step fails, then you should remove it and run the build manually (see below)
    - name: Autobuild
      uses: github/codeql-action/autobuild@v1

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
