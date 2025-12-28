## More About Me – [Take a Look!](http://www.mjakaria.me)

### SonarQube

SonarQube is an open-source platform used to continuously inspect and improve the quality of source code.

### Introduction

Modern software delivery is not just about releasing features fast. It is about delivering software `quickly`, `safely`, and `reliably`.

In this lecture, you will learn a complete `DevSecOps` workflow using `Maven`, `Jenkins`, `Docker`, and `SonarQube`. You will see how code quality and security checks are built directly into the CI pipeline, not added at the end.

The session explains how security shifts left in the SDLC, how SonarQube performs static code analysis (SAST), quality checks, and test coverage analysis, and how a real CI job can automatically fail when code does not meet defined quality or security standards.

By the end of this lecture, you will understand not only how to integrate SonarQube, but also why modern engineering teams design DevSecOps pipelines this way.

### Pre-requisites

To understand the full DevSecOps and SonarQube flow, the following sessions are recommended. They will make the final CI/CD pipeline example much easier to follow.

1. Modern SDLC Explained | Build Automation & Real-World Insights
2. CI/CD Explained | How Pipelines Work & Branching Strategies

### Install

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install openjdk-17-jdk -y
java -version
```

#### Configure Linux Kernel Limits

```bash
sudo vi /etc/sysctl.conf
# Add
vm.max_map_count=524288
fs.file-max=131072
# Check
sudo sysctl -p
```

#### User limits

```bash
sudo vi /etc/security/limits.conf
# Add
sonarqube   -   nofile   131072
sonarqube   -   nproc    8192
```

#### Install PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib -y
```

#### Create SonarQube DB & User

```bash
sudo -i -u postgres
```

```bash
psql
```

```bash
CREATE DATABASE sonarqube;
CREATE USER sonar WITH ENCRYPTED PASSWORD 'Sql@054003';
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonar;
\q
```

#### Create SonarQube User

```bash
sudo useradd -m -d /opt/sonarqube -s /bin/bash sonarqube
```

#### Download & Install SonarQube

```bash
cd /opt
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.3.79811.zip
sudo apt install unzip -y
sudo unzip sonarqube-9.9.3.79811.zip
sudo mv sonarqube-9.9.3.79811 sonarqube
sudo chown -R sonarqube:sonarqube /opt/sonarqube
```

#### Configure SonarQube Database

```bash
sudo vi /opt/sonarqube/conf/sonar.properties
```

```bash
sonar.jdbc.username=sonar
sonar.jdbc.password=Sql@054003
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube

sonar.web.host=0.0.0.0
sonar.web.port=9000
```

```bash
sudo ufw allow 9000/tcp
sudo ufw reload
```

#### Create systemd Service

```bash
sudo vi /etc/systemd/system/sonarqube.service
```

```bash
# For linux-x86-64 image
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonarqube
Group=sonarqube
Restart=always
LimitNOFILE=131072
LimitNPROC=8192

[Install]
WantedBy=multi-user.target
```

```bash
# For macosx-universal-64
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/macosx-universal-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/macosx-universal-64/sonar.sh stop
User=sonarqube
Group=sonarqube
Restart=always
LimitNOFILE=131072
LimitNPROC=8192

[Install]
WantedBy=multi-user.target
```

#### Start SonarQube

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable sonarqube
sudo systemctl start sonarqube
sudo systemctl status sonarqube
```

#### Access SonarQube

```bash
http://<server-ip>:9000
Username: `admin`
Password: `admin`
```

### What is DevSecOps and why it is important

#### What is DevSecOps?

DevSecOps stands for Development, Security, and Operations. It is an approach where security is built into every stage of the software development lifecycle (SDLC) instead of being added at the end.

##### How DevSecOps Works

| Traditional approach                                     | DevSecOps approach                              |
| -------------------------------------------------------- | ----------------------------------------------- |
| Code → Build → Test → Deploy → Security Check (too late) | Code → Build → Test →  Security Checks → Deploy |

#### Why DevSecOps Is Important

- Finds Security Issues Early
- Improves Code Quality
- Faster & Safer Releases
- Reduces Risk in Production
- Automation Instead of Manual Checks - (SAST, SCA, DAST)
- Shared Responsibility

### Common Tools Used in DevSecOps

| Area                      | Tools                              |
| ------------------------- | ---------------------------------- |
| CI/CD                     | Jenkins, GitHub Actions, GitLab CI |
| Code Quality & SAST       | SonarQube                          |
| Dependency Security (SCA) | OWASP Dependency-Check             |
| Containers                | Docker                             |
| Orchestration             | Kubernetes                         |
| IaC                       | Terraform, Ansible                 |

### Key DevSecOps terms and concepts

Before working with DevSecOps tools, it is important to understand the `main types of security and quality checks used in CI/CD pipelines`. Each check serves a different purpose and runs at a different stage of the pipeline. Knowing `what each one does and where it fits` helps you design effective and meaningful pipelines.

#### The most common DevSecOps check categories are

- `SAST` – Code security
- `DAST` – Runtime security
- `SCA` – Dependency risk
- `Code Quality` – Maintainability and correctness

> Important:
`SAST`, `DAST`, and `SCA` are not tools. They are methods or categories of checks. Tools like `SonarQube`, `OWASP ZAP`, or `Snyk` implement these methods. Always think method first, then choose the right tool.

#### SAST (Static Application Security Testing)

SAST analyzes an application’s source code, configuration, or binaries without running the application.

- Detects issues such as SQL injection risks, hardcoded credentials, insecure APIs, and unsafe coding patterns
- Runs early in development and CI pipelines
- Helps developers fix security issues before deployment

> Where it runs:

- IDEs and CI pipelines (during build time)

> Popular SAST tools:

- SonarQube, Checkmarx, Fortify SCA, Snyk Code, GitHub CodeQL, Semgrep

#### DAST (Dynamic Application Security Testing)

DAST tests an application while it is running, similar to how an attacker would interact with it.

- Sends real HTTP requests to find vulnerabilities like XSS, injection, authentication issues
- Does not require access to source code
- Commonly used in staging or pre-production environments

> Where it runs:

- Against a deployed application

> Popular DAST tools:

- OWASP ZAP, Burp Suite, Invicti (Netsparker), Acunetix, AppSpider

#### Code Quality Checks

Code quality checks focus on maintainability and correctness, not just security.

- They identify:
  - Long or complex methods
  - Duplicate logic
  - Poor naming
  - Dead or unused code

> Code smells

- Good code quality makes systems easier to maintain, test, and extend, and reduces future risk.

> Popular tools:

- SonarQube, SonarLint, ESLint, Pylint, Flake8, PMD, Checkstyle, RuboCop

#### Dependency Scanning (SCA – Software Composition Analysis)

SCA analyzes third-party libraries and frameworks used in an application.

- Detects known vulnerabilities and license issues
- Compares dependency versions against vulnerability databases
- Critical because modern applications rely heavily on open-source components

> Key distinction:

- SAST → vulnerabilities in your code
- SCA → vulnerabilities in your dependencies

> Popular SCA tools:

- Snyk, OWASP Dependency-Check, GitHub Dependabot, GitLab Dependency Scanning, Mend (WhiteSource), JFrog Xray, Trivy, Grype

#### SBOM (Software Bill of Materials)

An SBOM is a machine-readable inventory of:

- All components
- Versions
- Licenses used in an application, container image, or OS layer.
- Generated during CI
- Attached to build artifacts
- Enables supply-chain transparency and post-release rescanning

> Popular tools and formats:

- Syft, CycloneDX, SPDX

#### SCA vs SBOM

| Aspect                  | SCA (Software Composition Analysis)                | SBOM (Software Bill of Materials)             |
| ----------------------- | -------------------------------------------------- | --------------------------------------------- |
| Purpose                 | Detects security and license risks                 | Lists all components and versions             |
| Type                    | **Active scanning and analysis**                   | **Passive inventory**                         |
| What it does            | Scans dependencies against vulnerability databases | Creates a machine-readable list of components |
| Vulnerability detection | Yes                                                | No                                            |
| Policy enforcement      | Yes (can block builds)                             | No                                            |
| CI/CD impact            | Can fail pipelines                                 | Attached as an artifact                       |
| When used               | During build and CI pipeline                       | During or after build                         |
| CVE handling            | Finds known vulnerabilities immediately            | Enables future rescans when new CVEs appear   |
| Example tools           | Snyk, OWASP Dependency-Check, Mend                 | Syft, CycloneDX, SPDX                         |

> One-line takeaway

👉 SCA = active risk detection and policy enforcement
👉 SBOM = inventory for traceability and future analysis

### What is SonarQube and Where It Fits in CI/CD

SonarQube is a platform used for continuous code inspection, static analysis, and quality governance. It automatically analyzes source code (or compiled artifacts) on every commit to find:

- Bugs
- Security vulnerabilities (SAST)
- Code smells
- Code duplication
- Test coverage gaps

In simple terms, SonarQube turns code quality and security checks into an automated, continuous process that runs on every repository and every pipeline—long before code reaches production.

### SonarQube’s Role in CI/CD

When integrated into a CI/CD pipeline, SonarQube acts as a quality and security gate.

- Every build must pass SonarQube checks before packaging, deployment, or release
- Issues are detected early (shift left), directly in developer workflows
- Quality gates enforce measurable standards, not opinions

For DevOps and platform teams, SonarQube provides central visibility into code health across projects, ensuring consistency, compliance, and traceability.

### SonarQube’s `Three` Core Pillars

#### 1. Code Coverage

Code coverage measures how much of your source code is executed by automated tests. For example, if tests `execute 80 out of 100` lines, `coverage is 80%`.

> Practical tip

- Use coverage to find untested areas, but remember: high coverage does not guarantee good tests.

> Common coverage tools

- Java: JaCoCo, Cobertura
- JavaScript: LCOV / Istanbul
- Python: coverage.py

#### 2. Code Security (SAST)

SonarQube performs Static Application Security Testing (SAST) by analyzing code without running it. It detects:

- Vulnerabilities
- Security hotspots
- Security-sensitive bugs

Why it matters

- Issues are caught in pull requests or CI, not production
- Findings are categorized by severity for easy prioritization
- Helps teams meet security SLAs automatically

#### 3. Code Quality (Maintainability & Reliability)

Code can work correctly but still be poorly written.

SonarQube detects:

- Code smells
- Duplicate logic
- Overly complex or hard-to-read code

These issues make systems:

- Harder to understand
- Riskier to change
- Slower to evolve over time

### SonarQube Issue Types

#### 1. Bug

A bug is a defect in the code that can cause incorrect behavior or runtime failures, like

- Null pointer dereference
- Incorrect conditional logic
- Resource leaks (files or connections not closed)
- Concurrency or threading issues

#### 2. Code Smell

A code smell indicates poor design or low maintainability, even if the code works correctly today, like

- Very long methods
- Deeply nested conditionals
- Magic numbers
- Large classes doing too many things

#### 3. Duplication

Duplication means the same or very similar code appears in multiple places, often due to copy-paste, like

- Repeated business logic
- Similar validation code across classes

#### 4. Vulnerability

A vulnerability is a security weakness that can be exploited by an attacker, like

- SQL injection
- Cross-site scripting (XSS)
- Hardcoded credentials
- Unsafe deserialization
- Weak cryptographic algorithms

#### 5. Security Hotspot

A security hotspot is code that is security-sensitive and requires human review. The code is not always wrong. It becomes risky if used incorrectly, like

- Use of cryptographic APIs
- Manual SQL query construction
- Authentication or authorization configuration

### End-to-End Production DevSecOps Flow (Node.js + npm)

#### 1. Checkout Source Code

- Pipeline pulls source code, `package.json`, `package-lock.json`, and `Dockerfile` from a private Git repository
- Triggered by:
  - Pull requests
  - Merges to protected branches

#### 2. Install Dependencies & Test (CI Phase)

- Install dependencies `npm install`
- Run test `npm test`
- Test frameworks: `Jest`, `Mocha`, `Vitest`

#### 3. Code Coverage Generation, Coverage tools

- Istanbul / nyc (most common)
- Built into Jest `npm test -- --coverage`
- Generates coverage reports (LCOV/JSON)

#### 4. Run SCA & Generate SBOM

- Dependency Scanning (SCA)
  - Scan `package.json` and `package-lock.json`
  - Detect known vulnerabilities and license risks
- Common tools
  - Snyk
  - npm audit
  - OWASP Dependency-Check `npm audit --audit-level=high`
  - Can fail pipeline based on severity policy

#### 5. SonarQube Analysis (Static), SonarQube performs

- SAST
- Code quality checks
- Duplication detection
- Coverage enforcement (from LCOV)
- Typical run

```bash
sonar-scanner \
  -Dsonar.projectKey=node-app \
  -Dsonar.sources=. \
  -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
```

- Quality Gate, Enforces:
  - Minimum coverage on new code
  - No new critical bugs or vulnerabilities

- Pipeline behavior
  - Gate fails → pipeline stops
  - Gate passes → packaging continues

#### 6. Build & Containerize

- Build production bundle (if applicable) `npm run build`
- Build Docker image `docker build -t node-app:${GIT_SHA} .`

#### 7. Push Image & SBOM to Registry

- Push Docker image to registry (ECR, GCR, Docker Hub)
- Push SBOM alongside the image using ORAS or Cosign

#### 8. Deploy to Staging

- Deploy image to staging environment
- Mirrors production setup as closely as possible

#### 9. Run DAST & Smoke Tests - DAST

- Tools:
  - OWASP ZAP
  - Burp Suite

- Test running endpoints for:
  - XSS
  - Injection flaws
  - Auth issues
  - Smoke tests `curl /health`

#### 10. Promote to Production

- Promote only when:
  - SonarQube Quality Gate passes
  - DAST and smoke tests pass

- Deployment strategy
  - Canary rollout
  - Then full rollout

#### 11. Continuous Monitoring & SBOM Rescans

- Monitor production using: Logs, metrics, tracing
- Periodically rescan stored SBOMs against new CVEs

## With Regards, `Jakir`

[![LinkedIn][linkedin-shield-jakir]][linkedin-url-jakir]
[![Facebook-Page][facebook-shield-jakir]][facebook-url-jakir]
[![Youtube][youtube-shield-jakir]][youtube-url-jakir]

### Wishing you a wonderful day! Keep in touch

<!-- Personal profile -->

[linkedin-shield-jakir]: https://img.shields.io/badge/linkedin-%230077B5.svg?style=for-the-badge&logo=linkedin&logoColor=white
[linkedin-url-jakir]: https://www.linkedin.com/in/jakir-ruet/
[facebook-shield-jakir]: https://img.shields.io/badge/Facebook-%231877F2.svg?style=for-the-badge&logo=Facebook&logoColor=white
[facebook-url-jakir]: https://www.facebook.com/jakir.ruet/
[youtube-shield-jakir]: https://img.shields.io/badge/YouTube-%23FF0000.svg?style=for-the-badge&logo=YouTube&logoColor=white
[youtube-url-jakir]: https://www.youtube.com/@mjakaria-ruet/featured
