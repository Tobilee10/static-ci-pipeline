# TITLE: Production-Ready Dockerized Website CI Pipeline

A automated Continuous Integration (CI) pipeline built with GitHub Actions to lint, containerize, and security-scan a static website.

This repository demonstrates modern DevOps engineering practices, focusing on code quality enforcement, automated build caching, and shifting security left by running vulnerability scans before publishing artifacts to a registry.

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
└── dockerised-static-webiste-pipeline/
    ├── index.html                  # Main website landing page
    ├── styles.css                  # Core application stylesheets
    ├── Dockerfile
    ├── nginx.conf
    ├── .htmlhintrc
    ├── .styleintrc.json
    └── Readme.md
  


## 🏗️ Pipeline Architecture & Workflow

The workflow is broken down into three sequential stages, enforced by dependency rules (needs), ensuring that failing code never touches production.

[ Push to main ] ──> 1. Lint (HTML/CSS) ──> 2. Build & Scan (Trivy) locally ──> 3. Publish (Docker Hub)

1. **Code Quality (Linting)**

    Validates HTML structure using HTMLHint.

    Enforces modern style standards across stylesheets using Stylelint paired with stylelint-config-standard.

2. **Security-First Build (build-scan-push)**

    Generates a unique, incremental version tag tied directly to the execution pipeline (v1.${{ github.run_number }}).

    Builds the image locally using Docker Buildx and caches steps for rapid execution.

    Leverages Trivy to scan the local image image for CRITICAL or HIGH vulnerabilities. If vulnerabilities are found, the pipeline fails instantly and aborts the push.

    Upon validation, pushes both the :latest and the unique immutable version tag to Docker Hub.


## 🔐 Setup & Repository Secrets

To replicate this setup, you must configure the following GitHub Actions Secrets under your repository settings (Settings -> Secrets and variables -> Actions):
|Secret Name |Description |	Example Value
|------------|---------|-------------
DOCKER_USERNAME |	Your Docker Hub account username | my-docker-user
|DOCKER_PASSWORD |	Docker Hub Personal Access Token (PAT) |	dckr_pat_...

## 🚀 Getting Started

    Clone the repository:
    Bash

    git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
    cd YOUR_REPO_NAME

2. **Ensure your configuration files exist:**
   Make sure you have a `Dockerfile` configuring Nginx to serve your static assets, along with a `.stylelintrc` file for styling alignment.
3. **Trigger the workflow:**
   Commit a change and push it straight to your production branch:
   ```bash
   git add .
   git commit -m "feat: optimize pipeline orchestration"
   git push origin main

    Monitor the Run: Head over to the Actions tab of your GitHub repository to watch the multi-stage visual pipeline execute.