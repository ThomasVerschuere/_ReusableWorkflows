# Playwright .NET Dockerfile Template

This template Dockerfile packages Playwright .NET test projects for deployment to Azure Container Registry.

## Quick Start

**Just copy and customize!**

1. **Copy the workflow template:**
   ```bash
   cp templates/playwright-docker-acr-workflow.yml .github/workflows/playwright-docker.yml
   ```

2. **Copy the Dockerfile:**
   ```bash
   cp templates/Dockerfile ./Dockerfile
   ```

3. **Copy .dockerignore (optional but recommended):**
   ```bash
   cp templates/.dockerignore ./.dockerignore
   ```

4. **Update the TODO values** in `.github/workflows/playwright-docker.yml` (lines 15-16)

5. **Request ACR access** from Thomas Verschuere (see step 3 below) or configure your own ACR secrets

6. **Push and verify** the workflow runs successfully

**Reference example:** [SLC-RT-DaaS repository](https://github.com/SkylineCommunications/SLC-RT-DaaS)

---

## Quick Start Checklist

- [ ] Your project has `dotnetPlaywright.sh` and `config.json` in the project folder
- [ ] Copy `templates/playwright-docker-acr-workflow.yml` to `.github/workflows/playwright-docker.yml`
- [ ] Copy `templates/Dockerfile` to repository root
- [ ] Copy `templates/.dockerignore` to repository root (optional)
- [ ] Update `project-path` and `image-name` in the workflow file (lines 15-16)
- [ ] Request ACR access from Thomas or configure your own ACR secrets
- [ ] Push and verify workflow runs successfully

**Reference example:** [SLC-RT-DaaS repository](https://github.com/SkylineCommunications/SLC-RT-DaaS)

## Prerequisites

Before using this workflow, your project must have:

- ✅ A Playwright .NET test project (e.g., `YourProject.LiveServiceTesting`)
- ✅ `dotnetPlaywright.sh` - Bash script that runs your tests
- ✅ `config.json` - Test configuration file
- ✅ Azure Container Registry credentials (or request LiveServiceTests ACR access)

## Usage with `Playwright Docker ACR Workflow.yml`

### 1. Copy template files to your repository

```bash
# Copy workflow template
cp templates/playwright-docker-acr-workflow.yml .github/workflows/playwright-docker.yml

# Copy Dockerfile (no customization needed!)
cp templates/Dockerfile ./Dockerfile

# Copy .dockerignore (optional but recommended)
cp templates/.dockerignore ./.dockerignore
```

**No Dockerfile customization required!** The workflow automatically passes your `project-path` to the Dockerfile as a build argument.

### 2. Customize your workflow file

Open `.github/workflows/playwright-docker.yml` and update lines 15-16 (between the separator lines):

```yaml
project-path: 'YourProject.LiveServiceTesting'  # Your LiveServiceTesting project folder
image-name: 'your-tests'                        # Your ACR image name (lowercase only)
```

That's it! Everything else can stay as-is.

**Note:** Once organization secrets are configured, you can simplify the `secrets:` section to just `secrets: inherit`.

### Workflow Triggers

The template defaults to **manual trigger only** (`workflow_dispatch`). You can customize based on your needs:

**Manual only (default):**
```yaml
on:
  workflow_dispatch:
```

**Automatic on every push to main:**
```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:
```

**On pull requests (for testing before merge):**
```yaml
on:
  pull_request:
  workflow_dispatch:
```

**On releases:**
```yaml
on:
  release:
    types: [published]
  workflow_dispatch:
```

**Scheduled (e.g., nightly):**
```yaml
on:
  schedule:
    - cron: '0 2 * * *'  # Every day at 2 AM UTC
  workflow_dispatch:
```

**Image tagging:**  
The workflow pushes images tagged as `latest` by default to minimize ACR storage usage. Each push overwrites the previous image.

Need different tagging strategies (test/qa/prod)? Contact **Thomas Verschuere** ([email](mailto:thomas.verschuere@skyline.be) | [Teams](https://teams.microsoft.com/l/chat/0/0?users=thomas.verschuere@skyline.be)) to discuss `image-tag` configuration options.

### 3. Request ACR access

**To get started, contact Thomas Verschuere:**
- 📧 Email: [thomas.verschuere@skyline.be](mailto:thomas.verschuere@skyline.be)
- 💬 Teams: [Start chat](https://teams.microsoft.com/l/chat/0/0?users=thomas.verschuere@skyline.be)

After your request is approved, the following secrets will be provisioned to your repository:

- `LIVESERVICETESTS_ACR_LOGIN_SERVER` - Azure Container Registry login server (e.g., `liveservicetests-xyz.azurecr.io`)
- `LIVESERVICETESTS_ACR_USERNAME` - ACR username  
- `LIVESERVICETESTS_ACR_PASSWORD` - ACR password

**Note:** A self-service request form is being developed. Until then, reach out directly.

**Until you receive provisioned secrets,** use your own ACR credentials with explicit mapping (shown in step 3 above).

**After secrets are provisioned,** you can simplify your workflow to use `secrets: inherit` instead of explicit mapping.

## Project structure requirements

Your project must have this structure:

```
YourRepo/
├── .github/
│   └── workflows/
│       └── playwright-docker.yml
├── Dockerfile                              # This template, customized
├── .dockerignore                           # (Optional) Excludes unnecessary files from build
├── YourProject.LiveServiceTesting/
│   ├── YourProject.LiveServiceTesting.csproj
│   ├── dotnetPlaywright.sh                 # Test execution script
│   └── config.json                         # Test configuration
```

## How it works

1. **GitHub Actions workflow** calls the reusable workflow
2. **Reusable workflow** publishes your test project to `publish/tests`
3. **Docker builds** using this Dockerfile:
   - Copies published test binaries from `publish/tests`
   - Copies `dotnetPlaywright.sh` and `config.json` from your project folder
   - Pre-installs Playwright browsers
   - Sets up environment variables
4. **Image is pushed** to ACR with tags: `latest` and `{commit-sha}`

## Customization options

### Update Playwright version

Change the base image version:

```dockerfile
FROM mcr.microsoft.com/playwright/dotnet:v1.56.0-noble
```

### Add additional files

If your tests need extra configuration files:

```dockerfile
COPY YourProject.LiveServiceTesting/extra-config.json /app/extra-config.json
```

### Modify environment variables

Add or modify ENV directives as needed:

```dockerfile
ENV YOUR_CUSTOM_VAR=value
```

## Based on

This template is derived from the `SLC-RT-DaaS` reference implementation.

## Troubleshooting

### Workflow fails with "dotnet publish" error
- ✅ Verify `project-path` input matches your actual folder name
- ✅ Ensure the `.csproj` file exists at `{project-path}/{project-path}.csproj`

### Docker build fails with "no such file or directory" on COPY
- ✅ Check `ARG PROJECT_NAME` matches your folder name exactly
- ✅ Verify `dotnetPlaywright.sh` and `config.json` exist in your project folder

### Docker login fails
- ✅ Verify secrets are configured in repository settings
- ✅ Check secret names match exactly: `LIVESERVICETESTS_ACR_LOGIN_SERVER`, etc.
- ✅ Ensure ACR credentials are valid (test with `docker login` locally)

### Image pushes but doesn't run tests
- ✅ Check `dotnetPlaywright.sh` has Unix line endings (LF, not CRLF)
- ✅ Verify `config.json` is valid JSON
- ✅ Ensure `TESTFOLDERPATH` environment variable points to `/app/tests/`

**Need help?** Contact **Thomas Verschuere** ([email](mailto:thomas.verschuere@skyline.be) | [Teams](https://teams.microsoft.com/l/chat/0/0?users=thomas.verschuere@skyline.be))
