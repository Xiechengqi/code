
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
