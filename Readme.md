# TITLE: Production-Ready Dockerized Website CI Pipeline

This repository demonstrate an automated Continuous Integration Pipeline (CI) built with GitHub Actions to lint, containerize, and security-scan a static website, focusing on code quality enforcement, automated build, caching, and shifting security left by running vulnerability scan before publishing artifactx to Docker Hub registry.

## 🛠️ Tech Stack & Tools

- [x] CI Platform: GitHub Actions
- [x] Containerization: Docker & Docker Buildx
- [x] Web Server: Nginx (Internal container routing)
- [x] Security Scanning: Aqua Security Trivy
- [x] Container Registry: Docker Hub

## 📁 Repository Structure

├── .github/
│   └── workflows/
│       └── deploy.yml              # GitHub Actions pipeline configuration
├── app
│   ├── css
│   │   └── styles.css
│   └── index.html
├── Dockerfile
├── .htmlhintrc
├── nginx.conf
├── Readme.md
└── .stylelintrc.json
  


## 🏗️ Pipeline Architecture & CI Workflow

The workflow in .github/workflows/ci.yml is broken down into three sequential stages, enforced by dependency rules (needs), ensuring that failing code never touches production through the following steps:


[ Push to main ] ──> 1. Lint (HTML/CSS) ──> 2. Build & Scan (Trivy) locally ──> 3. Publish (Docker Hub)

1. **Code Quality (Linting)**

   - Validates HTML structure using HTMLHint.

   - Enforces modern style standards across stylesheets using Stylelint paired with stylelint-config-standard.

2. **Security-First Build (build-scan-push)**

   - Generates a unique, incremental version tag tied directly to the execution pipeline **(v1.${{ github.run_number }})**.

   - Builds the image locally using Docker Buildx and caches steps for rapid execution.

   - Trivy scan the local image for CRITICAL or HIGH vulnerabilities. If vulnerabilities are found, the pipeline fails instantly and aborts the push.

   - Upon validation, pushes both the :latest and the unique immutable version tag to Docker Hub.


## 🔐 Setup & Repository Secrets

To replicate this setup, you must configure the following GitHub Actions Secrets under your repository settings (Settings -> Secrets and variables -> Actions):

|Secret Name |Description |	Example Value
|------------|---------|-------------
DOCKER_USERNAME |	Your Docker Hub account username | my-docker-user
|DOCKER_PASSWORD |	Docker Hub Personal Access Token (PAT) |	dckr_pat_...

## 🚀 Run Locally

1. **Clone the repository:**
   ```bash

    git clone https://github.com/Tobilee10/static-ci-pipeline.git

    cd YOUR_REPO_NAME
    

2. **Ensure your configuration files exist:**
   Make sure you have a `Dockerfile` configuring Nginx to serve your static assets, along with a `.stylelintrc` file for styling alignment.
3. **Trigger the workflow:**
   Commit a change and push it straight to your production branch:
   ```bash
   git add .
   git commit -m "feat: optimize ci pipeline orchestration"
   git push origin main

    Monitor the Run: Head over to the Actions tab of your GitHub repository to watch the multi-stage visual pipeline execute.

## Challenges Faced:

Docker Hub personal access token (Read and Write Permission)

Lint style.css error

Trivy Version Not found

The Docker build fails because COPY in my Dockerfile (lines 5–6) references files that are not in the build context: index.html and styles.css.
