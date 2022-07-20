# github-actions-examples
 
## CodeQL

CodeQL - semantic code analysis. It is needed to find security vulnerabilities. CodeQL runs an extensible set of queries that have been developed by the community and the GitHub security lab to find common vulnerabilities in your code.

**uses:**
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
        language: [ 'python' ]
    steps:
    - name: Checkout repository
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
