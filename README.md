# 🛡️ Unified Code Analysis Security Scanning with AccuKnox GitHub Action

The **AccuKnox Code Analysis GitHub Action** is a single, unified action that runs any combination of AccuKnox ASPM code-analysis scans — **SAST, SCA, Secret, IaC, ML Static Scan, API Discovery, and SBOM** — and uploads the results to the **AccuKnox Console** for centralized visibility, risk tracking, and remediation.

Instead of wiring up a separate action for every scanner, configure **one step**, pick the scans you need via `scan_type`, and shift security left across your entire codebase — **before it reaches production**.

---

## 🎯 Key Features

- ✅ **7 Scanners, One Action** – SAST (OpenGrep), SCA (Trivy), Secret (TruffleHog), IaC (Checkov), ML Static Scan (ModelScan), API Discovery (code2api), and SBOM (image + filesystem).
- 🧩 **Run Any Combination** – Select one or many scans with a single comma/space separated `scan_type` input.
- ⌨️ **Command Text Per Scan** – Every scanner exposes a `<type>_command` input mapped directly to the CLI's `--command`, so you control exactly what each tool runs.
- 🏗️ **IaC with Frameworks** – Restrict IaC scans to one or more frameworks (e.g. `Kubernetes,Terraform`).
- 📦 **SBOM for Image & Filesystem** – Generate a CycloneDX SBOM from a container image or your source tree.
- 🔒 **Shift Left Security** – Integrate all checks directly into your CI/CD pipeline for early detection.
- 📥 **Seamless AccuKnox Console Integration** – Findings flow automatically to the AccuKnox dashboard.

---

## ⚠️ Prerequisites

Before using this GitHub Action, ensure the following are in place:

- 🔐 **AccuKnox Console Access** – Sign in to your AccuKnox tenant.
- 🗝️ **API Token** – Retrieve this from the AccuKnox Console (see Token Generation).
- 🏷️ **Label Created in Console** – For tagging the uploaded scan reports.
- 🔑 **GitHub Secrets Configured** – Store the required credentials securely in your repository's GitHub Secrets.

---

## 📌 Installation & Usage

### Step 1: Retrieve AccuKnox Credentials

1. Log in to your AccuKnox Console.
2. Navigate to **Settings → Tokens**.
3. Click **Create Token** and save the value as `Accuknox_token`.
4. Create a label under **Dashboard → Labels** to tag scan results.

---

### Step 2: Configure GitHub Secrets

Go to your repository: **Settings → Secrets and variables → Actions → New repository secret**, and add:

| Secret Name | Description |
|-------------|-------------|
| `ACCUKNOX_TOKEN`    | Your AccuKnox API token for authentication |
| `ACCUKNOX_ENDPOINT` | The AccuKnox API URL (e.g., `cspm.demo.accuknox.com`) |
| `ACCUKNOX_LABEL`    | Label used to tag and group scan results |

---

### Step 3: Define Your GitHub Workflow

Create a workflow file (e.g., `.github/workflows/accuknox-code-analysis.yml`) and add:

```yaml
name: AccuKnox Code Analysis Workflow

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  code-analysis:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v7.0.0

      - name: Run AccuKnox Code Analysis
        uses: accuknox/accuknox-code-analysis@latest
        with:
          # Pick any combination of scans
          scan_type: "sast, sca, secret, iac, ml, api-discovery, sbom"

          # AccuKnox credentials
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}

          # Common options
          soft_fail: true            # Continue the pipeline on findings
          upload_artifact: true      # Upload kept result files as an artifact

          # SBOM – image or filesystem (project_name required)
          sbom_scan_type: "image"
          sbom_image_ref: "myapp:latest"
          sbom_project_name: "my-project"
```

> 💡 Only the inputs for the scans listed in `scan_type` are used — everything else is ignored, so you can keep your workflow minimal.

---

## 📝 Examples

Each example below is a complete, copy-paste workflow. The **per-scanner** examples (one for each `scan_type`) show the minimal set of inputs for a single scan; the **unified** example runs everything in one step.

All examples assume the following secrets are configured: `ACCUKNOX_TOKEN`, `ACCUKNOX_ENDPOINT`, `ACCUKNOX_LABEL`.

### 1. SAST

```yaml
name: AccuKnox SAST
on:
  push:
    branches: [main]
jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7.0.0
      - name: Run AccuKnox SAST
        uses: accuknox/accuknox-code-analysis@latest
        with:
          scan_type: "sast"
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
          sast_severity: "HIGH,CRITICAL"
          soft_fail: true
```

### 2. SCA

```yaml
name: AccuKnox SCA
on:
  push:
    branches: [main]
jobs:
  sca:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7.0.0
      - name: Run AccuKnox SCA
        uses: accuknox/accuknox-code-analysis@latest
        with:
          scan_type: "sca"
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
          sca_severity: "HIGH,CRITICAL"
          soft_fail: true
```

### 3. Secret

```yaml
name: AccuKnox Secret Scan
on:
  push:
    branches: [main]
jobs:
  secret:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7.0.0
        with:
          fetch-depth: 0   # full history for git-based secret scanning
      - name: Run AccuKnox Secret Scan
        uses: accuknox/accuknox-code-analysis@latest
        with:
          scan_type: "secret"
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
          soft_fail: true
```

### 4. IaC

```yaml
name: AccuKnox IaC
on:
  push:
    branches: [main]
jobs:
  iac:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7.0.0
      - name: Run AccuKnox IaC Scan
        uses: accuknox/accuknox-code-analysis@latest
        with:
          scan_type: "iac"
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
          soft_fail: true
```

### 5. ML Static Scan 

```yaml
name: AccuKnox ML Static Scan
on:
  push:
    branches: [main]
jobs:
  ml:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7.0.0
      - name: Run AccuKnox ML Static Scan
        uses: accuknox/accuknox-code-analysis@latest
        with:
          scan_type: "ml"
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
          soft_fail: true
```

### 6. API Discovery (Note: WIP)

```yaml
name: AccuKnox API Discovery
on:
  push:
    branches: [main]
jobs:
  api-discovery:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7.0.0
      - name: Run AccuKnox API Discovery
        uses: accuknox/accuknox-code-analysis@latest
        with:
          scan_type: "api-discovery"
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
          soft_fail: true
```

### 7. SBOM

> **Prerequisite — Create a Project.** To associate SBOM data with the correct entity, you must create a **Project** in the AccuKnox Console first.
>
> 1. Log in to the **AccuKnox Dashboard**.
> 2. Navigate to **SBOM → Projects**.
> 3. Click **New Project**.
> 4. Fill in the required details:
>    - **Name\*** – Project name (use this value as `sbom_project_name` in the workflow).
>    - **Description** – Short description of the project.
>    - **Classifier\*** – Select **Container** for an **image** SBOM, or **Application** for a **filesystem** SBOM.
>    - **Tags** – (Optional) Add relevant tags.
> 5. Click **Create**.

```yaml
name: AccuKnox SBOM
on:
  push:
    branches: [main]
jobs:
  sbom:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7.0.0
      - name: Run AccuKnox SBOM (filesystem)
        uses: accuknox/accuknox-code-analysis@latest
        with:
          scan_type: "sbom"
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
          sbom_scan_type: "filesystem"
          sbom_scan_path: "."
          sbom_project_name: "my-project"   # required for SBOM
          soft_fail: true
```

> For an **image** SBOM, set `sbom_scan_type: "image"` and `sbom_image_ref: "myapp:latest"` (build/pull the image earlier in the same job).

### 8. Unified — All Scans in One Step

```yaml
name: AccuKnox Code Analysis (Unified)
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
jobs:
  code-analysis:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7.0.0

      - name: Run AccuKnox Code Analysis
        uses: accuknox/accuknox-code-analysis@latest
        with:
          # Run every scanner in a single step
          scan_type: "sast, sca, secret, iac, ml, sbom"

          # AccuKnox credentials
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}

          # Common options
          soft_fail: true
          upload_artifact: true

          # SAST – override default severity (HIGH)
          sast_severity: "HIGH,CRITICAL"

          # SBOM (project_name required)
          sbom_scan_type: "filesystem"
          sbom_scan_path: "."
          sbom_project_name: "my-project"
```

---

## ⚙️ Configuration Options (Inputs)

### Common

| Input | Description | Optional/Required | Default |
|-------|-------------|-------------------|---------|
| `scan_type` | Scans to run (comma/space separated): `sast`, `sca`, `secret`, `iac`, `ml`, `api-discovery`, `sbom` | Required | — |
| `accuknox_token` | API token for authenticating with AccuKnox SaaS | Required | — |
| `accuknox_endpoint` | URL of the AccuKnox Console to push results | Required | — |
| `accuknox_label` | Label used in AccuKnox SaaS to organise and identify results | Required | — |
| `scanner_version` | Git tag of the `accuknox-aspm-scanner` binary to download (must include `sca`, `ml`, `api-discovery`) | Optional | `v0.14.7-rc.3` |
| `soft_fail` | Prevent CI from failing on findings (applies to all scans) | Optional | `true` |
| `upload_artifact` | Upload kept result files (`results*.json`, `results*.jsonl`) as a GitHub artifact | Optional | `false` |

### SAST (`sast`)

| Input | Description | Optional/Required | Default |
|-------|-------------|-------------------|---------|
| `sast_command` | Command text passed to `--command` (target to scan) | Optional | `.` |
| `sast_severity` | Comma-separated severities (`LOW, MEDIUM, HIGH, CRITICAL`) | Optional | `HIGH` |

### SCA (`sca`)

| Input | Description | Optional/Required | Default |
|-------|-------------|-------------------|---------|
| `sca_command` | Command text passed to `--command` (e.g. `fs .`) | Optional | `fs .` |
| `sca_severity` | Comma-separated severities to fail on | Optional | `""` |

### Secret (`secret`)

| Input | Description | Optional/Required | Default |
|-------|-------------|-------------------|---------|
| `secret_command` | Command text passed to `--command` (e.g. `git file://.` or `filesystem .`) | Optional | `git file://.` |
| `secret_additional_arguments` | Extra arguments appended to the command (e.g. `--results verified --branch main`) | Optional | `""` |

### IaC (`iac`)

| Input | Description | Optional/Required | Default |
|-------|-------------|-------------------|---------|
| `iac_command` | Raw command text passed to `--command`. When set, overrides the structured inputs below | Optional | `""` |
| `iac_directory` | Directory with infrastructure code to scan | Optional | `.` |
| `iac_file` | Specific file to scan; cannot be used with `iac_directory` | Optional | `""` |
| `iac_framework` | One or more frameworks (comma-separated), e.g. `Kubernetes,Terraform` | Optional | `""` (all) |
| `iac_compact` | Do not display code blocks in output | Optional | `true` |
| `iac_quiet` | Display only failed checks | Optional | `true` |

### ML Static Scan (`ml`)

| Input | Description | Optional/Required | Default |
|-------|-------------|-------------------|---------|
| `ml_command` | Command text passed to `--command` (e.g. `scan -p . -r json`) | Optional | `scan -p . -r json` |
| `ml_model_name` | Custom collector/model identifier | Optional | `""` |
| `ml_source_type` | Source type for metadata | Optional | `github` |

### API Discovery (`api-discovery`)

| Input | Description | Optional/Required | Default |
|-------|-------------|-------------------|---------|
| `api_command` | Command text passed to `--command` (e.g. `-path . -output results.json`) | Optional | `-path . -output results.json` |

### SBOM (`sbom`)

| Input | Description | Optional/Required | Default |
|-------|-------------|-------------------|---------|
| `sbom_scan_type` | Target type: `image` or `filesystem` | Optional | `filesystem` |
| `sbom_image_ref` | Image reference (required when `sbom_scan_type` is `image`) | Optional | `""` |
| `sbom_scan_path` | Filesystem path (used when `sbom_scan_type` is `filesystem`) | Optional | `.` |
| `sbom_command` | Raw command text passed to `--command`. When set, overrides the structured inputs above | Optional | `""` |
| `sbom_severity` | Comma-separated severities | Optional | `""` |
| `sbom_project_name` | Project name (AccuKnox entity). **Required** for SBOM | Optional* | `""` |

\* Required only when `sbom` is included in `scan_type`.

---

## 🔍 How It Works

1. **Developer pushes code** – A push or pull request triggers the GitHub Action.
2. **Scanner setup (once)** – The action validates credentials, parses `scan_type`, and downloads the `accuknox-aspm-scanner` binary for the requested `scanner_version`.
3. **Selected scans run** – Each enabled scan executes in `--command` mode inside a container, building its arguments from your `<type>_command` and scan-specific inputs:
   - **SAST** → OpenGrep static analysis
   - **SCA** → Trivy dependency/composition analysis
   - **Secret** → TruffleHog secret detection
   - **IaC** → Checkov misconfiguration checks (optionally per framework)
   - **ML** → ModelScan static ML model analysis
   - **API Discovery** → code2api route/endpoint discovery
   - **SBOM** → CycloneDX bill of materials for an image or filesystem
4. **Results uploaded to AccuKnox Console** – Using the provided `accuknox_token` and `accuknox_label`.
5. **Optional artifact upload** – If `upload_artifact: true`, kept result files are saved as a GitHub artifact.
6. **Review findings** – Available in AccuKnox Console: **Dashboard → Issues → Findings**, filtered by scan type.
7. **Pipeline decision** – If `soft_fail: false`, the pipeline fails when findings are detected.

---

## 📖 Support & Documentation

- 📚 **Read More:** [AccuKnox Docs](https://help.accuknox.com/integrations/github-overview/)
- 📧 **Contact Support:** support@accuknox.com

---

## 🏁 Conclusion

The **AccuKnox Code Analysis GitHub Action** consolidates seven security scanners into a single, configurable step — letting you run SAST, SCA, Secret, IaC, ML, API Discovery, and SBOM scans together, surface findings in the AccuKnox Console, and enforce policy gates across your CI/CD pipeline.

**🔐 Shift Left with AccuKnox – Secure Your Code from Commit to Cloud! ☁️🛡️**

