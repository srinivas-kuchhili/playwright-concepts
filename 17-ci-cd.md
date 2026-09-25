# 17 — CI/CD Integration

## 1. CI Essentials

| Concern | Setting |
| ------- | ------- |
| Headless | default `headless: true` |
| Install browsers + OS deps | `npx playwright install --with-deps` |
| Retries | `retries: process.env.CI ? 2 : 0` |
| Workers | `workers: process.env.CI ? 2 : undefined` (or 50%) |
| Block `test.only` | `forbidOnly: !!process.env.CI` |
| Artifacts | upload `playwright-report/` & `test-results/` |
| Report | `html` + `junit` (for CI test tab) / `blob` for shards |
| Secrets | CI secret store → env vars |
| Consistency | Official Docker image `mcr.microsoft.com/playwright` |

---

## 2. GitHub Actions

```yaml
# .github/workflows/playwright.yml
name: Playwright Tests
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * *'          # nightly regression
  workflow_dispatch:
    inputs:
      tag:
        description: 'Tag to run (e.g. @smoke)'
        default: '@smoke'

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: lts/*
          cache: npm
      - name: Install dependencies
        run: npm ci
      - name: Install Playwright browsers
        run: npx playwright install --with-deps
      - name: Run Playwright tests
        run: npx playwright test --grep "${{ github.event.inputs.tag || '' }}"
        env:
          BASE_URL: ${{ vars.BASE_URL }}
          APP_USER: ${{ secrets.APP_USER }}
          APP_PASS: ${{ secrets.APP_PASS }}
      - uses: actions/upload-artifact@v4
        if: ${{ !cancelled() }}
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

### Sharded GitHub Actions + merged report

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shardIndex: [1, 2, 3, 4]
        shardTotal: [4]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: lts/* }
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test --shard=${{ matrix.shardIndex }}/${{ matrix.shardTotal }}
      - uses: actions/upload-artifact@v4
        if: ${{ !cancelled() }}
        with:
          name: blob-report-${{ matrix.shardIndex }}
          path: blob-report
          retention-days: 1

  merge-reports:
    if: ${{ !cancelled() }}
    needs: [test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: lts/* }
      - run: npm ci
      - uses: actions/download-artifact@v4
        with:
          path: all-blob-reports
          pattern: blob-report-*
          merge-multiple: true
      - run: npx playwright merge-reports --reporter html ./all-blob-reports
      - uses: actions/upload-artifact@v4
        with:
          name: html-report--attempt-${{ github.run_attempt }}
          path: playwright-report
          retention-days: 14
```

(Config: `reporter: process.env.CI ? 'blob' : 'html'`)

---

## 3. Jenkins (Declarative Pipeline)

```groovy
pipeline {
  agent {
    docker {
      image 'mcr.microsoft.com/playwright:v1.55.0-noble'
      args '--ipc=host'
    }
  }
  parameters {
    choice(name: 'ENV', choices: ['qa', 'staging'], description: 'Environment')
    string(name: 'TAG', defaultValue: '@smoke', description: 'Test tag')
  }
  environment {
    CI = 'true'
    APP_CREDS = credentials('app-credentials')   // APP_CREDS_USR / APP_CREDS_PSW
  }
  stages {
    stage('Install') { steps { sh 'npm ci' } }
    stage('Test') {
      steps { sh "ENV=${params.ENV} npx playwright test --grep ${params.TAG}" }
    }
  }
  post {
    always {
      junit 'results/junit.xml'
      publishHTML(target: [
        reportDir: 'playwright-report', reportFiles: 'index.html',
        reportName: 'Playwright Report', keepAll: true, alwaysLinkToLastBuild: true
      ])
      archiveArtifacts artifacts: 'test-results/**', allowEmptyArchive: true
    }
  }
}
```

---

## 4. Azure DevOps

```yaml
trigger: [main]
pool: { vmImage: ubuntu-latest }
steps:
  - task: NodeTool@0
    inputs: { versionSpec: '20.x' }
  - script: npm ci
  - script: npx playwright install --with-deps
  - script: npx playwright test
    env:
      CI: 'true'
      APP_PASS: $(APP_PASS)
  - task: PublishTestResults@2
    condition: succeededOrFailed()
    inputs:
      testResultsFormat: JUnit
      testResultsFiles: results/junit.xml
  - task: PublishPipelineArtifact@1
    condition: succeededOrFailed()
    inputs:
      targetPath: playwright-report
      artifact: playwright-report
```

Also: **Azure Playwright Workspaces / Microsoft Playwright Testing** runs browsers in the cloud with high parallelism.

---

## 5. GitLab CI

```yaml
playwright:
  image: mcr.microsoft.com/playwright:v1.55.0-noble
  stage: test
  script:
    - npm ci
    - npx playwright test
  artifacts:
    when: always
    paths: [playwright-report/, test-results/]
    reports:
      junit: results/junit.xml
    expire_in: 1 week
```

---

## 6. Docker

```dockerfile
FROM mcr.microsoft.com/playwright:v1.55.0-noble
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["npx", "playwright", "test"]
```

```bash
docker build -t pw-tests .
docker run --rm --ipc=host -v "$(pwd)/playwright-report:/app/playwright-report" pw-tests
```

💡 Keep the Docker image tag **identical** to your `@playwright/test` version.
💡 `--ipc=host` avoids Chromium crashes due to limited shared memory.

---

## 7. CI Strategy (what to say)

```text
PR / commit     → lint + @smoke on chromium (fast feedback, < 10 min)
Merge to main   → full regression, all browsers, sharded
Nightly         → full regression + visual + cross-browser + mobile
Release         → sanity on staging/prod
```

> **Interview one-liner:** In CI I install with `--with-deps` or use the official Playwright Docker image, run headless with retries and limited workers, forbid `test.only`, shard across a matrix and merge blob reports, publish JUnit to the CI test tab and upload the HTML report + traces as artifacts. Credentials come from CI secrets. Smoke runs on PRs, full regression nightly.
