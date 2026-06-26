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
    

2. **Ensure your configuration files exist:**
   Make sure you have a `Dockerfile` configuring Nginx to serve your static assets, along with a `.stylelintrc` file for styling alignment.
3. **Trigger the workflow:**
   Commit a change and push it straight to your production branch:
   ```bash
   git add .
   git commit -m "feat: optimize ci pipeline orchestration"
   git push origin main

    Monitor the Run: Head over to the Actions tab of your GitHub repository to watch the multi-stage visual pipeline execute.
## Image Pushed successfully into Docker Hub Repository
![success](/static-ci-pipeline/images/docker-hub-success.png)

## Challenges Faced & Solution:

<details>
<summary><b>1. Docker Build Context Mismatch (failed to calculate checksum)</b></summary>

- **The Challenge:** While setting up the automated CI deployment pipeline, the GitHub Actions build failed during the Docker build stage with a `failed to calculate checksum:... not found` error for `index.html` and `styles.css`.
![](/static-ci-pipeline/images/docker-error3.png)
- **The Cause:** The repository was structured with application files inside an `app/` subfolder. Because the Docker build execution was triggered from the root repository directory, the `Dockerfile`'s default `COPY index.html .` and `COPY styles.css` command looked in the wrong location.
- **The Solution:** Adjusted the build context configuration within the `Dockerfile` by updating the source paths (`COPY app/index.html .`) and `COPY app/css/styles.css .` to correctly map relative to the project root directory.
</details>

<details>
<summary><b>2. Git Divergent Branches Warning</b></summary>

- **The Challenge:** Encountered a blocking Git warning regarding divergent local and remote branches (`reconcile them... before your next pull`) when syncing local updates.
- **The Solution:** Configured a clear pull strategy preference (`git config pull.rebase true`) to maintain a clean, linear commit history before pulling the changes down safely.
</details>


<details>
<summary><b>3. Docker Hub Personal Access Token (PAT) Authentication Failures</b></summary>

- **The Challenge:** Automated GitHub Actions workflows failed during the docker login or push stages, returning `Permission Denied` errors despite utilizing the correct credentials.
![](/static-ci-pipeline/images)
- **The Cause:** The Docker Hub Personal Access Token (PAT) used in GitHub Secrets was initially generated with restricted or `Read-Only` permissions, preventing the CI/CD pipeline from pushing newly built images to the registry.
- **The Solution:** Regenerated the Personal Access Token in the Docker Hub account settings, explicitly assigning it **Read & Write** permissions, and updated the corresponding repository secret in GitHub.
</details>


<details>
<summary><b>4. Stylelint Configuration Errors during CSS Linting</b></summary>

- **The Challenge:** The CSS linting job in the CI pipeline failed to execute or threw formatting errors because it lacked standard rulesets for evaluating `styles.css`.
![](/static-ci-pipeline/images/lint-css-error.png)
- **The Cause:** The workflow could not find a local runtime configuration file specifying the structural and syntax expectations for the codebase.
- **The Solution:** Created and deployed a `.stylelintrc.json` configuration file at the application level, extending standard community rulesets (`stylelint-config-standard`) to enforce uniform style properties across the stylesheet.
</details>


<details>
<summary><b>5. Trivy Vulnerability Scan Failures (Version Not Found)</b></summary>

- **The Challenge:** Security analysis phases using the Aquasecurity Trivy action crashed mid-execution with an error indicating a specific tool version or database reference was `not found`.
![](/static-ci-pipeline/images/trivy-error.png)
- **The Cause:** The workflow file was referencing an outdated, deprecated, or hardcoded version tag of the Trivy GitHub Action that no longer aligned with the current remote vulnerability databases.
- **The Solution:** Updated the workflow step to reference the latest stable major release of the official GitHub Action (`aquasecurity/trivy-action@master`), ensuring smooth image parsing and up-to-date vulnerability fetching.
</details>


