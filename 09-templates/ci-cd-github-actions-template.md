# CI/CD GitHub Actions Pipeline Template

> Copy this file to `.github/workflows/ci.yml` in your product repository. It automates all mandatory quality, architecture, security, and testing gates defined by the Engineering Playbook.

```yaml
name: CI Quality Gate & Build

on:
  push:
    branches: [ main, release/* ]
  pull_request:
    branches: [ main, release/* ]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # =========================================================================
  # 1. Security & Secrets Pre-Check
  # =========================================================================
  security-scan:
    name: Secret & Vulnerability Scan
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Source
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: TruffleHog Secret Scanner
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD

  # =========================================================================
  # 2. Backend Gate: Checkstyle (800 lines), ArchUnit, Unit & Integration Tests
  # =========================================================================
  backend-gate:
    name: Backend Build & ArchUnit Tests
    runs-on: ubuntu-latest
    needs: security-scan
    steps:
      - name: Checkout Source
        uses: actions/checkout@v4

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: 'maven'

      - name: Verify Checkstyle (800-line File Limit & Code Quality)
        run: mvn checkstyle:check -Dcheckstyle.config.location=checkstyle.xml

      - name: Run Unit & ArchUnit Architecture Tests
        run: mvn test -Dtest="*Test,ArchitectureTest"

      - name: Run Testcontainers Integration Tests (MySQL & Redis)
        run: mvn test -Dtest="*IT"

  # =========================================================================
  # 3. Frontend Gate: TypeScript, ESLint (400 lines), Tests
  # =========================================================================
  frontend-gate:
    name: Frontend Quality & Linter
    runs-on: ubuntu-latest
    needs: security-scan
    steps:
      - name: Checkout Source
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json

      - name: Install Dependencies
        working-directory: ./frontend
        run: npm ci

      - name: TypeScript Strict Check
        working-directory: ./frontend
        run: npm run type-check

      - name: ESLint (File Line Limits, Restricted Imports & a11y)
        working-directory: ./frontend
        run: npm run lint

      - name: Run Frontend Tests
        working-directory: ./frontend
        run: npm run test:ci

  # =========================================================================
  # 4. Packaging: Immutable Container Build & Trivy Scan
  # =========================================================================
  package-and-scan:
    name: Immutable Image Build & Container Scan
    runs-on: ubuntu-latest
    needs: [ backend-gate, frontend-gate ]
    steps:
      - name: Checkout Source
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build Immutable Image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          push: false
          tags: product-api:${{ github.sha }}
          load: true

      - name: Scan Image with Trivy (Vulnerabilities & Misconfigurations)
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: product-api:${{ github.sha }}
          format: 'table'
          exit-code: '1'
          ignore-unfixed: true
          severity: 'CRITICAL,HIGH'
```
