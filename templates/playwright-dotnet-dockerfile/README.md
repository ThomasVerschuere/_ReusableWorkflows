# Playwright .NET Dockerfile Template

This template Dockerfile packages Playwright .NET test projects for deployment to Azure Container Registry.

## Quick Start Checklist

- [ ] Your project has `dotnetPlaywright.sh` and `config.json` in the project folder
- [ ] Copy `Dockerfile` to repository root (no customization needed!)
- [ ] Copy `.dockerignore` to repository root (optional)
- [ ] Create `.github/workflows/playwright-docker.yml`
- [ ] Configure ACR secrets (your own or request LiveServiceTests access)
- [ ] Push and verify workflow runs successfully

**Reference example:** [SLC-RT-DaaS repository](https://github.com/SkylineCommunications/SLC-RT-DaaS)

## Prerequisites

Before using this workflow, your project must have:

- ✅ A Playwright .NET test project (e.g., `YourProject.LiveServiceTesting`)
- ✅ `dotnetPlaywright.sh` - Bash script that runs your tests
- ✅ `config.json` - Test configuration file
- ✅ Azure Container Registry credentials (or request LiveServiceTests ACR access)

## Usage with `Playwright Docker ACR Workflow.yml`

### 1. Copy template files to your repository root

```bash
# Copy Dockerfile (no customization needed!)
cp templates/playwright-dotnet-dockerfile/Dockerfile ./Dockerfile

# Copy .dockerignore (optional but recommended)
cp templates/playwright-dotnet-dockerfile/.dockerignore ./.dockerignore
```

**No Dockerfile customization required!** The workflow automatically passes your `project-path` to the Dockerfile as a build argument.

### 2. Create your workflow file

Create `.github/workflows/playwright-docker.yml`:

```yaml
name: Build and Push Playwright Tests

on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:

jobs:
  build-and-push:
    uses: SkylineCommunications/_ReusableWorkflows/.github/workflows/Playwright Docker ACR Workflow.yml@main
    with:
      # TODO: Update with your project folder name (e.g., MyApp.LiveServiceTesting)
      project-path: 'YourProject.LiveServiceTesting'
      
      # TODO: Update with your desired image name (lowercase only, e.g., myapp-tests)
      image-name: 'your-tests'
      
      # Usually leave these as-is unless you have specific requirements
      dockerfile-path: 'Dockerfile'
      dotnet-version: '8.x'
    secrets:
      # Map your ACR secrets to the expected names
      LIVESERVICETESTS_ACR_LOGIN_SERVER: ${{ secrets.YOUR_ACR_LOGIN_SERVER }}
      LIVESERVICETESTS_ACR_USERNAME: ${{ secrets.YOUR_ACR_USERNAME }}
      LIVESERVICETESTS_ACR_PASSWORD: ${{ secrets.YOUR_ACR_PASSWORD }}
```

**What you need to customize:**
- ✏️ `project-path` - Your LiveServiceTesting project folder name
- ✏️ `image-name` - Your desired ACR image name (must be lowercase)
- ✏️ Secret values in your repository settings

**What you can leave as-is:**
- ✅ `dockerfile-path: 'Dockerfile'` (unless you renamed it)
- ✅ `dotnet-version: '8.x'` (unless using different version)
- ✅ The `uses:` line pointing to the reusable workflow
- ✅ Trigger conditions (on push/PR/workflow_dispatch)

**Once you have been provisioned the LiveServiceTests ACR secrets** (after contacting Thomas), simplify the secrets section:
```yaml
jobs:
  build-and-push:
    uses: SkylineCommunications/_ReusableWorkflows/.github/workflows/Playwright Docker ACR Workflow.yml@main
    with:
      project-path: 'YourProject.LiveServiceTesting'  # TODO: Update this
      image-name: 'your-tests'                        # TODO: Update this
      dockerfile-path: 'Dockerfile'
      dotnet-version: '8.x'
    secrets: inherit  # ✅ Automatically passes LIVESERVICETESTS_ACR_* secrets
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
