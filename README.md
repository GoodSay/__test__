# github-actions-examples 
- [CodeQL](#codeql)
- [Flake8](#flake8)
- [Build and test](#build-and-test)
- [Dependency review](#dependency-review)
- [Remote SSH commands](#remote-ssh-commands)
- [Deploy](#deploy)
- [Test pytest](#pytest)
- [Test unittest](#unittest)


## CodeQL

CodeQL - semantic code analysis. It is needed to find security vulnerabilities. CodeQL runs an extensible set of queries that have been developed by the community and the GitHub security lab to find common vulnerabilities in your code.

**uses:**
- Checkout source repository: `actions/checkout@v3`
- Initialize CodeQL: `github/codeql-action/init@v2`
- Autobuild CodeQL: `github/codeql-action/autobuild@v2`
- Perform CodeQL Analysis: `github/codeql-action/analyze@v2`

**CodeQL supports:**  
cpp, csharp, go, java, ruby, javascript, python

**default-file-name:** `.github/workflows/codeql.yml`
```yml
name: CodeQL
on:
  push:
    branches:
      - 'master'
      - 'dev'
  pull_request:
    branches:
      - 'master'
      - 'dev'
  schedule:
    - cron: '0 2,4 * * 0'
jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest
    permissions:
      actions: read
      contents: read
      security-events: write
    strategy:
      fail-fast: false
      matrix:
        language: ['python']
    steps:
    - name: Checkout source repository
      uses: actions/checkout@v3
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v2
      with:
        languages: ${{ matrix.language }}
    - name: Autobuild CodeQL
      uses: github/codeql-action/autobuild@v2
    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v2
```

## Flake8

Flake8 - a tool that detects to scan the project code and features in it for stylistic errors and violations of various Python code conventions.

**uses:**
- Checkout source repository: `actions/checkout@v3`
- Setup Python environment: `actions/setup-python@v4`
- Flake8 Lint: `py-actions/flake8@v2`

**default-file-name:** `.github/workflows/flake8.yml`
```yml
name: Flake8
on:
  push:
    branches:
      - 'master'
      - 'dev'
  pull_request:
    branches:
      - 'master'
      - 'dev'
jobs:
  flake8:
    runs-on: ubuntu-latest
    name: Flake8 lint
    steps:
      - name: Checkout source repository
        uses: actions/checkout@v3
      - name: Setup Python environment
        uses: actions/setup-python@v4
        with:
          python-version: "3.10"
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install flake8
      - name: Flake8 Lint
        uses: py-actions/flake8@v2      
```

## Build and test

**uses:**
- Checkout source repository: `actions/checkout@v3`
- Setup Python: `actions/setup-python@v4`
- Upload Test Results: `actions/upload-artifact@v2`
- Download Artifacts: `actions/download-artifact@v2`
- Publish Test Results: `EnricoMi/publish-unit-test-result-action@v1`

**max-parallel:**
value from 1 up to 10

**operating systems:**
- macos: macos-12, macos-latest, macos-11
- windows: windows-latest, windows-2022 windows-2019, windows-2016
- linux: ubuntu-22.04, ubuntu-latest, ubuntu-20.04, ubuntu-18.04

**python-version:**
- old: 3.9, 3.8, 3.7, 3.6
- use: 3.10, 3.11-dev, 3.11.0-alpha.1

**default-file-name:** `.github/workflows/build-and-test-full.yml`
```yml
name: Action
on:
  push:
    branches:
      - 'master'
      - 'dev'
  pull_request:
    branches:
      - 'master'
      - 'dev'
permissions:
  contents: read
  issues: read
  checks: write
  pull-requests: write
jobs:
  build-and-test:
    name: Build (python-${{ matrix.python-version }}, ${{ matrix.os }})
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      max-parallel: 4
      matrix:
        os: [macos-latest, windows-2022, windows-2019, ubuntu-18.04, ubuntu-20.04, ubuntu-22.04, ubuntu-latest]
        python-version: ["3.8", "3.9", "3.10", "3.11-dev", "3.11.0-alpha.1"]
    steps:
      - name: Checkout source repository
        uses: actions/checkout@v3
      - name: Setup Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pytest
          pip install -r requirements.txt
      - name: PyTest
        run: python -m pytest test --junit-xml pytest.xml
      - name: Upload Test Results
        if: always()
        uses: actions/upload-artifact@v2
        with:
          name: Test Results (Python ${{ matrix.python-version }})
          path: pytest.xml
  publish-results:
    name: Publish Tests Results
    needs: build-and-test
    runs-on: ubuntu-latest
    permissions:
      checks: write
      pull-requests: write
      contents: read
      issues: read
    if: always()
    steps:
      - name: Download Artifacts
        uses: actions/download-artifact@v2
        with:
          path: artifacts
      - name: Publish Test Results
        uses: EnricoMi/publish-unit-test-result-action@v1
        id: test-results
        if: always()
        with:
          files: "artifacts/**/*.xml"
```

## Dependency review

**uses:**
- Checkout source repository: `actions/checkout@v3`
- Dependency review: `actions/dependency-review-action@v2`

**fail-on-severity:**
critical, high, moderate, low

**licenses:**
- allow-licenses: MIT, Apache-2.0, GPL-3.0, BSD-3-Clause
- deny-licenses: Apache-1.1, LGPL-2.0, BSD-2-Clause

**default-file-name:** `.github/workflows/dependency-review.yml`
```yml
name: Dependency review
on: [pull_request]
permissions:
  contents: read
jobs:
  dependency-review:
    runs-on: ubuntu-latest
    steps:
      - name: 'Checkout source repository'
        uses: actions/checkout@v3
      - name: Dependency review
        uses: actions/dependency-review-action@v2
        with:
          fail-on-severity: low
          deny-licenses: Apache-1.1, LGPL-2.0, BSD-2-Clause
```

## Remote SSH commands

**uses:**
- Executing remote ssh commands using ssh key: `appleboy/ssh-action@master`

**params:**
- host (ip address)
- username (user)
- key (ssh-key)
- port (default: 22)
- script (bash command)

**default-file-name:** `.github/workflows/remote-ssh.yml`
```yml
name: Remote SSH commands
on:
  push:
    branches:
      - 'master'
      - 'dev'
  pull_request:
    branches:
      - 'master'
      - 'dev'
jobs:
  remote:
    runs-on: ubuntu-latest
    name: Remote SSH commands
    steps:
      - name: executing remote ssh commands using ssh key
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.KEY }}
          port: ${{ secrets.PORT }}
          script: whoami
```

## Deploy

**uses:**
- Checkout source repository: `actions/checkout@v3`
- Install SSH key: `shimataro/ssh-key-action@v2`

**params:**
- host (ip address)
- username (user)
- key (ssh-key)

**default-file-name:** `.github/workflows/deploy.yml`
```yml
name: Deploy
on:
  push:
    branches:
      - 'master'
jobs:
  deploy:
    runs-on: ubuntu-latest
    name: Deploy on ssh server
    steps:
      - name: Checkout source repository
        uses: actions/checkout@v3
      - name: Install SSH key
        uses: shimataro/ssh-key-action@v2
        with:
          key: ${{ secrets.KEY }} 
          known_hosts: ''
      - name: Adding Known Hosts
        run: ssh-keyscan -H ${{ secrets.HOST }} >> ~/.ssh/known_hosts
      - name: Deploy with rsync
        run: rsync -avz /home/runner/work/repository/repository/ ${{ secrets.USERNAME }}@${{ secrets.HOST }}:/home/destination
```

## Pytest

**default-file-name:** `.github/workflows/pytest.yml`
```yml
name: Pytest
on:
  push:
    branches:
      - 'master'
      - 'dev'
  pull_request:
    branches:
      - 'master'
      - 'dev'
jobs:
  pytest:
    name: Pytest
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python environment
        uses: actions/setup-python@v4
        with:
          python-version: "3.10"
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pytest
          pip install -r requirements.txt
      - name: PyTest
        run: python -m pytest test --junit-xml pytest.xml
```
      
## Unittest

**default-file-name:** `.github/workflows/unittest.yml`
```yml
name: Unittest
on:
  push:
    branches:
      - 'master'
      - 'dev'
  pull_request:
    branches:
      - 'master'
      - 'dev'
jobs:
  unittest:
    name: Unittest
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python environment
        uses: actions/setup-python@v4
        with:
          python-version: "3.10"
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
      - name: Test with unittest
        run: |
          python -m unittest discover tests/ '*_test.py'
```

## Pull request verify labels

**uses:**
- Checkout source repository: `actions/checkout@v3`
- Verify type labels: `zwaldowski/match-label-action@v2`

**default-file-name:** `.github/workflows/verify-type-labels.yml`
```yml
name: Verify type labels
on:
  pull_request:
    types: [opened, labeled, unlabeled, synchronize]
jobs:
  triage:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source repository
        uses: actions/checkout@v3
      - name: Verify type labels
        uses: zwaldowski/match-label-action@v2
        with:
          allowed: 'config, documentation, features, fix, hotfix, test'
```

## Auto release drafter

**uses:**
- Checkout source repository: `actions/checkout@v3`
- Update draft release: `release-drafter/release-drafter@v5`

**default-file-name:** `.github/workflows/release-draft.yml`
```yml
name: Create draft release
on:
  push:
    branches:
      - 'branch'
jobs:
  Update draft release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source repository
        uses: actions/checkout@v3
      - name: Update draft release
        uses: release-drafter/release-drafter@v5
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```


**default-file-name:** `.github/release-drafter.yml`
```yml
name-template: 'v$NEXT_PATCH_VERSION'
tag-template: 'v$NEXT_PATCH_VERSION'
categories:
  - title: ' New Features'
    labels:
      - 'features'
  - title: ' Bugs Fixes'
    labels:
      - 'fix'
      - 'hotfix'
  - title: ' Documentation'
    labels:
      - 'documentation'
  - title: ' Configuration'
    labels:
      - 'config'
  - title: ' Tests'
    labels:
      - 'test'

change-template: '- $TITLE @$AUTHOR (#$NUMBER)'
template: |
  ## Changes
  $CHANGES
```

## Pull request auto-label

**default-file-name:** `.github/labeler.yml`
```yml
# Add 'repo' label to any root file changes
repo:
- '*'

# Add 'test' label to any change to *.spec.js files within the source dir
test:
- src/**/*.spec.js

# Add 'source' label to any change to src files within the source dir EXCEPT for the docs sub-folder
source:
- any: ['src/**/*', '!src/docs/*']

# Add 'documentation` at all .md files
documentation:
- '*.md'

# Add 'frontend` label to any change to *.js files as long as the `main.js` hasn't changed
frontend:
- any: ['src/**/*.js']
  all: ['!src/main.js']
```

**uses:**
- Checkout source repository: `actions/checkout@v3`
- Pull Request Labeler: `actions/labeler@v4`

**default-file-name:** `.github/workflows/labeler.yml`
```yml
name: Pull Request Labeler
on:
- pull_request_target
jobs:
  triage:
    permissions:
      contents: read
      pull-requests: write
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source repository
        uses: actions/checkout@v3
      - name: Pull Request Labeler
        uses: actions/labeler@v4
        with:
          repo-token: "${{ secrets.GITHUB_TOKEN }}"
```

## Pull request reviewers

**default-file-name:** `.github/assignee.yml`
```yml
addReviewers: true
numberOfReviewers: 1
reviewers:
 - Zvezdolom

addAssignees: true
assignees:
 - Zvezdolom
numberOfAssignees: 1

skipKeywords:
  - draft
```

**uses:**
- Checkout source repository: `actions/checkout@v3`
- Auto assignment: `kentaro-m/auto-assign-action@v1.1.2`

**default-file-name:** `.github/workflows/assignee.yml`
```yml
name: 'Auto assign assignees or reviewers'
on: pull_request

jobs:
  add-reviews:
    name: Auto assignment
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source repository
        uses: actions/checkout@v3
      - name: Auto assignment
        uses: kentaro-m/auto-assign-action@v1.1.2
        with:
          configuration-path: ".github/assignee.yml"
```          

## Pull request auto merge

**uses:**
- **Checkout source repository:** `actions/checkout@v3`
- **Automerge:** `pascalgn/automerge-action@v0.12.0`

**default-file-name:** `.github/workflows/auto_merge.yml`
```yml
name: Auto merge
on:
  pull_request:
    types:
      - labeled
      - unlabeled
      - synchronize
      - opened
      - edited
      - ready_for_review
      - reopened
      - unlocked
  pull_request_review:
    types:
      - submitted
  check_suite:
    types:
      - completed
  status: {}
jobs:
  automerge:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source repository
        uses: actions/checkout@v3
      - name: 'Auto merge'
        uses: "pascalgn/automerge-action@v0.12.0"
        env:
          GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
          MERGE_METHOD: 'merge'
          MERGE_LABELS: "approved,!work"
          MERGE_REMOVE_LABELS: "approved"
          MERGE_COMMIT_MESSAGE: ""
          MERGE_RETRIES: "6"
          MERGE_RETRY_SLEEP: "10000"
          UPDATE_LABELS: ""
          UPDATE_METHOD: "merge"
          MERGE_DELETE_BRANCH: false
```
		  
## Pull request auto approve

**uses:**
- **Checkout source repository:** `actions/checkout@v3`
- **Label when approved:** `pullreminders/label-when-approved-action@master`

**default-file-name:** `.github/workflows/auto_approve.yml`
```yml
on: pull_request_review
name: 'Label approved pull requests'
jobs:
  labelWhenApproved:
    name: 'Label when approved'
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source repository
        uses: actions/checkout@v3
      - name: 'Label when approved'
        uses: pullreminders/label-when-approved-action@master
        env:
          APPROVALS: "1"
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          ADD_LABEL: "approved"
          REMOVE_LABEL: "awaiting review"
```

## Auto release drafter 2

**uses:**
**Checkout source repository:** `actions/checkout@v3`
**Update release draft:** `release-drafter/release-drafter@v5`

default-file-name: `.github/workflows/release-drafter.yml`
```yml
name: Release Drafter
on:
  push:
    branches:
      - master
jobs:
  update_release_draft:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source repository
        uses: actions/checkout@v3
      - name: Update release draft
        uses: release-drafter/release-drafter@v5
        with:
          publish: true
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**default-file-name:** `.github/release-draft.yml`
```yml
template: |
  ## Changes
  $CHANGES
change-template: '- **$TITLE** (#$NUMBER)'

version-template: "$MAJOR.$MINOR.$PATCH"
name-template: '$RESOLVED_VERSION'
tag-template: '$RESOLVED_VERSION'

categories:
  - title: 'Features'
    labels:
      - 'feature'
      - 'type:common'
  - title: 'Bug Fixes'
    labels:
      - 'fix'
      - 'bugfix'
      - 'bug'
      - 'hotfix'
      - 'dependencies'
  - title: 'Maintenance'
    labels:
      - 'build'
      - 'refactoring'
      - 'docs'
      - 'tests'

change-title-escapes: '\<*_&'

version-resolver:
  major:
    labels:
      - major
      - refactoring
  minor:
    labels:
      - feature
      - minor
      - common
  patch:
    labels:
      - patch
      - build
      - bug
      - bugfix
      - hotfix
      - fix
      - docs
      - tests
  default: patch
```

## badges

**uses:**
- Checkout source repository: `actions/checkout@v3`
- Create Awesome Badge: `schneegans/dynamic-badges-action@v1.4.0`

**default-file-name:** `.github/workflows/badge.yml`
```yml
name: Badge
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source repository
        uses: actions/checkout@v3
      - name: Create Awesome Badge
        uses: schneegans/dynamic-badges-action@v1.4.0
        with:
          auth: ${{ secrets.GIST }}
          gistID: ${{ secrets.GIST_ID }}
          filename: ${{ secrets.FILE_NAME }}
          label: Hello
          message: World
          color: orange
```
