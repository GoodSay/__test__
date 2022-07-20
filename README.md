# github-actions-examples
 
## CodeQL

CodeQL - semantic code analysis. It is needed to find security vulnerabilities. CodeQL runs an extensible set of queries that have been developed by the community and the GitHub security lab to find common vulnerabilities in your code.

**uses:**
- Checkout source repository: `actions/checkout@v3`
- Initialize CodeQL: `github/codeql-action/init@v2`
- Autobuild CodeQL: `github/codeql-action/autobuild@v2`
- Perform CodeQL Analysis: `github/codeql-action/analyze@v2`

**CodeQL supports:**  
cpp, csharp, go, java, ruby, javascript, python

```yml
name: CodeQL
on:
  push:
    branches:
      - 'main'
      - 'dev'
  pull_request:
    branches:
      - 'main'
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
- Set up Python environment: `actions/setup-python@v4`
- Flake8 Lint: `py-actions/flake8@v2`

```yml
name: Flake8
on:
  push:
    branches:
      - 'main'
      - 'dev'
  pull_request:
    branches:
      - 'main'
      - 'dev'
jobs:
  flake8:
    runs-on: ubuntu-latest
    name: Flake8 lint
    steps:
      - name: Checkout source repository
        uses: actions/checkout@v3
      - name: Set up Python environment
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
