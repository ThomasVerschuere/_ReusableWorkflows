# GitHub Actions publishes the test project before building this image.
# The runner handles the .NET build; Docker only packages the result.
FROM mcr.microsoft.com/playwright/dotnet:v1.56.0-noble

WORKDIR /app

# Test binaries published by the GitHub Actions workflow step
COPY publish/tests /app/tests/

# dotnetPlaywright.sh is not part of the publish output, copy it explicitly
# Strip Windows CRLF line endings so bash can execute it on Linux
# TODO: Replace {ProjectName} with your actual project folder name
COPY {ProjectName}/dotnetPlaywright.sh /app/tests/dotnetPlaywright.sh
RUN sed -i 's/\r//' /app/tests/dotnetPlaywright.sh

# Customer-specific config (one level above TESTFOLDERPATH, as expected by the script)
# TODO: Replace {ProjectName} with your actual project folder name
COPY {ProjectName}/config.json /app/config.json

# Create the output directory used by dotnet vstest --ResultsDirectory
RUN mkdir -p /app/outputs

# Pre-install the Playwright browsers for the exact version used by this project,
# so no browser download happens at container startup (saves time and bandwidth)
RUN cd /app/tests && pwsh playwright.ps1 install --with-deps

# TESTFOLDERPATH is read by dotnetPlaywright.sh; config.json is expected at ../config.json
ENV TESTFOLDERPATH=/app/tests/

# Tells dotnetPlaywright.sh to skip the playwright.ps1 install step at runtime
ENV SKIP_PLAYWRIGHT_INSTALL=true

ENTRYPOINT ["/bin/bash", "/app/tests/dotnetPlaywright.sh"]
