# CI/CD with Anypoint API Catalog CLI

[![Publish to Exchange](https://github.com/muleaceacademy/cicd-anypoint-catalog-cli/actions/workflows/maven.yml/badge.svg)](https://github.com/muleaceacademy/cicd-anypoint-catalog-cli/actions/workflows/maven.yml)

> **A complete CI/CD demonstration showing how to automatically publish API specifications to MuleSoft Anypoint Exchange using the Anypoint API Catalog CLI and GitHub Actions.**

---
## YouTube Tutorial

Watch the complete walkthrough here:

<a href="https://www.youtube.com/watch?v=WsrA3YZolME" target="_blank" rel="noopener noreferrer">
  <img 
    src="https://img.youtube.com/vi/WsrA3YZolME/maxresdefault.jpg" 
    alt="Watch the YouTube video" 
    width="720"
  />
</a>

--- 

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Repository Structure](#repository-structure)
- [Prerequisites](#prerequisites)
- [How It Works](#how-it-works)
  - [GitHub Actions Workflow](#github-actions-workflow)
  - [Catalog Descriptor](#catalog-descriptor)
  - [Exchange Metadata](#exchange-metadata)
- [API Specification](#api-specification)
- [Configuration](#configuration)
  - [GitHub Secrets Setup](#github-secrets-setup)
- [Running Locally](#running-locally)
- [CI/CD Pipeline Flow](#cicd-pipeline-flow)
- [References](#references)

---

## Overview

This repository demonstrates how to implement a **fully automated CI/CD pipeline** that publishes API assets (RAML specifications) to **MuleSoft Anypoint Exchange** using the [Anypoint API Catalog CLI](https://docs.mulesoft.com/api-catalog-cli/latest/).

Whenever a developer pushes changes to the `master` branch (or raises a Pull Request), GitHub Actions automatically:

1. Checks out the repository
2. Installs the `api-catalog-cli` tool
3. Publishes the API specification and its documentation to Anypoint Exchange

This pattern enables **API-first development** with automated asset governance ensuring that your API catalog on Anypoint Platform always reflects the latest approved version of your API specifications.

---

## Architecture

```
+---------------------------------------------------------------+
|                      GitHub Repository                        |
|                                                               |
|  +------------------+  +-------------+  +-----------------+   |
|  | API Spec (RAML)  |  | catalog     |  | home.md (docs)  |   |
|  | authors-restful  |  | .yaml       |  |                 |   |
|  +--------+---------+  +------+------+  +--------+--------+   |
|           |                   |                  |            |
|           +-------------------+------------------+            |
|                               |                               |
|                    git push / pull_request                    |
|                               |                               |
|          +--------------------v---------------------------+   |
|          |         GitHub Actions Workflow                |   |
|          |   Step 1: Checkout repo                        |   |
|          |   Step 2: Install api-catalog-cli (npm)        |   |
|          |   Step 3: api-catalog publish-asset            |   |
|          +--------------------+--------------------------+    |
+-------------------------------|-------------------------------+
                                |
                                v
                +-------------------------------+
                |  MuleSoft Anypoint Exchange   |
                |                               |
                |  [OK] API Asset Published     |
                |  [OK] Documentation Attached  |
                |  [OK] Metadata & Tags Applied |
                +-------------------------------+
```

---

## Repository Structure

```
cicd-anypoint-catalog-cli/
|
+-- .github/
|   +-- workflows/
|       +-- maven.yml              # GitHub Actions CI/CD pipeline
|
+-- authors-restful-api.raml       # RAML 1.0 API Specification
+-- catalog.yaml                   # Anypoint API Catalog Descriptor
+-- exchange.json                  # Anypoint Exchange Asset Metadata
+-- home.md                        # API Home Page (published to Exchange)
+-- README.md                      # This file
```

### File Descriptions

| File | Purpose |
|------|---------|
| `authors-restful-api.raml` | The RAML 1.0 specification defining the Authors RESTful API endpoints, data types, and examples |
| `catalog.yaml` | The **Catalog Descriptor** — tells `api-catalog-cli` how to publish the asset (version, tags, contact, documentation links) |
| `exchange.json` | Anypoint Exchange metadata including the organization ID, group ID, asset ID, and Design Center project references |
| `home.md` | The API home/landing page documentation published alongside the spec on Exchange |
| `.github/workflows/maven.yml` | The GitHub Actions workflow that automates the publish process on every push to `master` |

---

## Prerequisites

To use this repository and run the CI/CD pipeline, you need:

1. **MuleSoft Anypoint Platform account** with an active organization
2. **Anypoint Platform credentials** (username and password) with the `Exchange Contributors` permission
3. **GitHub repository** with the ability to configure Secrets
4. **Node.js 14+** (only required if running locally)
5. **Anypoint API Catalog CLI** (`api-catalog-cli`) — installed automatically by the workflow

---

## How It Works

### GitHub Actions Workflow

File: `.github/workflows/maven.yml`

The workflow is the heart of the CI/CD pipeline. It is triggered on push or pull request to the `master` branch:

```yaml
name: Publish to Exchange using Catalog CLI

on:
  push:
    branches: [ "master" ]
  pull_request:
    branches: [ "master" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Repo
      uses: actions/checkout@v3

    - name: Install Catalog-CLI
      run: |
        sudo apt-get update
        npm install -g api-catalog-cli@latest

    - name: Publish to Exchange
      env:
        PLATFORM_ORGID: ${{secrets.PLATFORM_ORGID}}
        PLATFORM_USERNAME: ${{secrets.PLATFORM_USERNAME}}
        PLATFORM_PASSWORD: ${{secrets.PLATFORM_PASSWORD}}
      run: |
        api-catalog publish-asset \
          --organization="$PLATFORM_ORGID" \
          --username "$PLATFORM_USERNAME" \
          --password "$PLATFORM_PASSWORD"
```

**Workflow Triggers:**

| Trigger | Description |
|---------|-------------|
| `push` to `master` | Publishes the latest API version to Anypoint Exchange |
| `pull_request` to `master` | Validates the API can be published before merge |

**Steps Explained:**

| Step | Description |
|------|-------------|
| **Checkout Repo** | Clones the repository contents into the GitHub Actions runner |
| **Install Catalog-CLI** | Installs the latest version of `api-catalog-cli` globally via npm |
| **Publish to Exchange** | Runs `api-catalog publish-asset` with Anypoint Platform credentials stored in GitHub Secrets |

---

### Catalog Descriptor

File: `catalog.yaml`

```yaml
#%Catalog Descriptor 1.0
projects:
  - main: authors-restful-api.raml
    assetId: authors-restful-api
    version: 1.0.1
    apiVersion: v1
    tags:
      - author
      - books
      - library
      - sys-api
    contact:
      name: 'Ashish P'
      email: 'ashish.pardhi@muleaceacademy.com'
    documentation:
      Home: home.md
```

The `catalog.yaml` is the **core configuration file** for the API Catalog CLI. It specifies:

| Field | Value | Description |
|-------|-------|-------------|
| `main` | `authors-restful-api.raml` | Entry point of the API specification |
| `assetId` | `authors-restful-api` | Unique identifier for the Exchange asset |
| `version` | `1.0.1` | Semantic version to publish to Exchange |
| `apiVersion` | `v1` | The API version label shown in Exchange |
| `tags` | `author`, `books`, `library`, `sys-api` | Searchable tags in Exchange |
| `contact.name` | `Ashish P` | Contact person for this API |
| `contact.email` | `ashish.pardhi@muleaceacademy.com` | Contact email |
| `documentation.Home` | `home.md` | Home page content published to Exchange |

> **Tip:** Update the `version` field in `catalog.yaml` before pushing to publish a new version of the asset to Exchange.

---

### Exchange Metadata

File: `exchange.json`

```json
{
  "main": "authors-restful-api.raml",
  "name": "Authors RESTful API",
  "organizationId": "52b545f8-286a-4227-a2b8-acc3a9df4588",
  "groupId": "52b545f8-286a-4227-a2b8-acc3a9df4588",
  "assetId": "authors-restful-api",
  "version": "1.0.0",
  "metadata": {
    "branchId": "master",
    "commitId": "fc6e75e6e96baf3cb2597cb7b284407bcb70c078",
    "projectId": "c2be2d70-7f0f-457c-80ef-a2d05b5b3cab"
  },
  "apiVersion": "v1",
  "classifier": "raml",
  "dependencies": [],
  "tags": [],
  "originalFormatVersion": "1.0"
}
```

| Field | Description |
|-------|-------------|
| `organizationId` | Your Anypoint Platform Organization ID |
| `groupId` | Business Group ID where the asset will be published |
| `assetId` | Unique asset identifier in Exchange |
| `classifier` | Asset type — `raml` for RAML API specifications |
| `metadata.projectId` | Linked Design Center project ID |

---

## API Specification

File: `authors-restful-api.raml`

This repository includes a sample **Authors RESTful API** specification written in RAML 1.0. It is used as the API asset that gets published to Anypoint Exchange through the CI/CD pipeline.

### API Details

| Property | Value |
|----------|-------|
| **Title** | Authors RESTful API - Salesforce Webinar UK April 2026 |
| **Version** | v1 |
| **Protocols** | HTTPS, HTTP |
| **Format** | RAML 1.0 |

### Data Type: `author`

| Property | Type | Required |
|----------|------|----------|
| `authorId` | string | Yes |
| `name` | string | Yes |
| `isbn` | string | Yes |
| `email` | string | Yes |
| `phone` | string | Yes |

### Endpoints

#### GET /authors

Returns a list of all authors.

- **Auth required:** No
- **Response:** `200 OK` — Array of `author` objects

Example Response:

```json
[
  {
    "authorId": "1",
    "name": "Ashish Pardhi",
    "email": "jeanette.p@mulesoft.com",
    "phone": "9781593275846",
    "isbn": "isbn-59327-asfn"
  },
  {
    "authorId": "2",
    "name": "Giavani Frediani",
    "email": "g.f@apisero.com",
    "phone": "9781593275846",
    "isbn": "isbn-12345-asfn"
  }
]
```

#### GET /authors/{authorId}

Returns details for a specific author by ID.

- **Auth required:** No
- **Path Parameter:** `authorId` — The unique ID of the author
- **Response:** `200 OK` — Single `author` object

Example Response:

```json
{
  "authorId": "1",
  "name": "Jeanette Penddreth",
  "email": "j.p@muesoft.com",
  "phone": "9781593275846",
  "isbn": "isbn-59327-asfn"
}
```

---

## Configuration

### GitHub Secrets Setup

The CI/CD pipeline uses three GitHub repository secrets to authenticate with Anypoint Platform. These must be configured before the workflow can run successfully.

**Steps to configure:**

1. Navigate to your GitHub repository
2. Click **Settings** tab
3. In the left sidebar, click **Secrets and variables** then **Actions**
4. Click **New repository secret** and add each of the following:

| Secret Name | Description | Where to Find |
|-------------|-------------|---------------|
| `PLATFORM_ORGID` | Anypoint Platform Organization ID | Anypoint Platform → Access Management → Organization → Organization ID |
| `PLATFORM_USERNAME` | Anypoint Platform account username (email) | Your Anypoint login email |
| `PLATFORM_PASSWORD` | Anypoint Platform account password | Your Anypoint login password |

> **Security Best Practice:** For production environments, consider using a **Connected App** (OAuth 2.0 client credentials) instead of username/password. See [Anypoint Connected Apps documentation](https://docs.mulesoft.com/access-management/connected-apps-overview) for details.

---

## Running Locally

You can run the API Catalog CLI locally to validate and publish your API before pushing to GitHub.

### Step 1: Install the CLI

```bash
# Install the Anypoint API Catalog CLI via npm
npm install -g api-catalog-cli@latest

# Verify installation
api-catalog --version
```

### Step 2: Validate Your Catalog Descriptor

```bash
# Navigate to your project directory
cd cicd-anypoint-catalog-cli

# Validate the catalog.yaml without publishing
api-catalog validate --organization=<YOUR_ORG_ID>
```

### Step 3: Publish the Asset

```bash
# Publish the API asset to Anypoint Exchange
api-catalog publish-asset \
  --organization="<YOUR_ORG_ID>" \
  --username "<YOUR_ANYPOINT_USERNAME>" \
  --password "<YOUR_ANYPOINT_PASSWORD>"
```

### Available CLI Commands

| Command | Description |
|---------|-------------|
| `api-catalog publish-asset` | Publish or update an API asset in Anypoint Exchange |
| `api-catalog validate` | Validate the catalog descriptor without publishing |
| `api-catalog --help` | Show all available commands and options |

---

## CI/CD Pipeline Flow

The following diagram describes the end-to-end flow from code change to Exchange publication:

```
Developer
   |
   | (1) Edit API spec or catalog.yaml
   |
   v
GitHub Repository
   |
   | (2) git push to master / open PR
   |
   v
GitHub Actions Runner (ubuntu-latest)
   |
   | (3) actions/checkout@v3
   |     --> Clones repo into runner workspace
   |
   | (4) npm install -g api-catalog-cli@latest
   |     --> Installs the Catalog CLI tool
   |
   | (5) api-catalog publish-asset
   |     --> Reads catalog.yaml
   |     --> Reads authors-restful-api.raml
   |     --> Reads home.md (documentation)
   |     --> Authenticates with Anypoint Platform
   |     --> Publishes asset to Exchange
   |
   v
Anypoint Exchange
   |
   | (6) Asset available at:
   |     https://anypoint.mulesoft.com/exchange/<orgId>/authors-restful-api/
   |
   v
API Consumers
   | - Browse in Exchange
   | - Import into API Manager
   | - Consume in Mule applications
```

### Version Management

Each time the `version` field in `catalog.yaml` is incremented and pushed to `master`, a **new version** of the asset is created in Exchange. Older versions remain accessible.

```
catalog.yaml version: 1.0.0  -->  Exchange: authors-restful-api v1.0.0
catalog.yaml version: 1.0.1  -->  Exchange: authors-restful-api v1.0.1
catalog.yaml version: 1.1.0  -->  Exchange: authors-restful-api v1.1.0
```

---

## References

- [MuleSoft Anypoint API Catalog CLI Documentation](https://docs.mulesoft.com/api-catalog-cli/latest/)
- [Anypoint Exchange Documentation](https://docs.mulesoft.com/exchange/)
- [RAML 1.0 Specification](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Anypoint Access Management - Connected Apps](https://docs.mulesoft.com/access-management/connected-apps-overview)
- [MuleSoft General Documentation](https://docs.mulesoft.com/general/)

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature-name`)
3. Update the API spec or catalog configuration
4. Commit your changes (`git commit -m "feat: describe your change"`)
5. Push to the branch (`git push origin feature/your-feature-name`)
6. Open a Pull Request against `master`

The GitHub Actions workflow will automatically validate the publish process when your PR is opened.

---

## License

This project is for educational and demonstration purposes. See [muleaceacademy](https://github.com/muleaceacademy) for more MuleSoft learning resources.
