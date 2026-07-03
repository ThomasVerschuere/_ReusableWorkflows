# Playwright .NET Dockerfile Template

This template Dockerfile packages Playwright .NET test projects for deployment to Azure Container Registry.

## Usage with `Playwright Docker ACR Workflow.yml`

### 1. Copy this Dockerfile to your repository root

```bash
cp templates/playwright-dotnet-dockerfile/Dockerfile ./Dockerfile
```

### 2. Customize the Dockerfile

Replace `{ProjectName}` placeholders (appears 2 times) with your actual project folder name:

```dockerfile
# Example: If your project is MyApp.LiveServiceTesting
COPY MyApp.LiveServiceTesting/dotnetPlaywright.sh /app/tests/dotnetPlaywright.sh
COPY MyApp.LiveServiceTesting/config.json /app/config.json
```

### 3. Create your workflow file

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
      project-path: 'YourProject.LiveServiceTesting'  # Your project folder name
      image-name: 'your-tests'                        # Desired ACR image name
      dockerfile-path: 'Dockerfile'
      dotnet-version: '8.x'
    secrets: inherit
```

**Image tagging:**  
The workflow pushes images tagged as `latest` by default to minimize ACR storage usage. Each push overwrites the previous image.

Need different tagging strategies (test/qa/prod)? Contact **Thomas Verschuere** ([email](mailto:thomas.verschuere@skyline.be) | [Teams](https://teams.microsoft.com/l/chat/0/0?users=thomas.verschuere@skyline.be)) to discuss `image-tag` configuration options.

### 4. Request ACR access

**To get started, contact Thomas Verschuere:**
- 📧 Email: [thomas.verschuere@skyline.be](mailto:thomas.verschuere@skyline.be)
- 💬 Teams: [Start chat](https://teams.microsoft.com/l/chat/0/0?users=thomas.verschuere@skyline.be)

After your request is approved, the following secrets will be provisioned to your repository:

- `LIVESERVICETESTS_ACR_REGISTRY` - Azure Container Registry URL
- `LIVESERVICETESTS_ACR_USERNAME` - ACR username  
- `LIVESERVICETESTS_ACR_PASSWORD` - ACR password

**Note:** A self-service request form is being developed. Until then, reach out directly.

**For SkylineCommunications repositories:** Once secrets are added, `secrets: inherit` in your workflow automatically passes them to the reusable workflow.

**Using your own ACR credentials?** Map your secrets explicitly:
```yaml
secrets:
  LIVESERVICETESTS_ACR_REGISTRY: ${{ secrets.YOUR_ACR_REGISTRY }}
  LIVESERVICETESTS_ACR_USERNAME: ${{ secrets.YOUR_ACR_USERNAME }}
  LIVESERVICETESTS_ACR_PASSWORD: ${{ secrets.YOUR_ACR_PASSWORD }}
```

## Project structure requirements

Your project must have this structure:

```
YourRepo/
├── .github/
│   └── workflows/
│       └── playwright-docker.yml
├── Dockerfile                              # This template, customized
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
