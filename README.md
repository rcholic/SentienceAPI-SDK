# Sentience API SDK

Automated SDK generation for Python and TypeScript/JavaScript clients.

## Repository Structure

```
.
├── openapi.json              # OpenAPI specification
├── openapitools.json         # OpenAPI Generator configuration
├── .github/workflows/
│   └── release-sdks.yml      # Automated SDK release workflow
└── sdks/                     # Generated SDKs (created during release)
    ├── python/               # Python SDK
    └── typescript/           # TypeScript SDK
```

## How It Works

When you create a new release in GitHub:
1. The workflow automatically generates Python and TypeScript SDKs from `openapi.json`
2. Packages are versioned based on your Git tag (e.g., `v1.0.0` → `1.0.0`)
3. Python package is published to PyPI as `sentienceapi-py`
4. TypeScript package is published to npm as `@rcholic/sentience-sdk`

## Setup Instructions

### 1. Configure PyPI Token

1. Go to [PyPI.org](https://pypi.org) → Account Settings → API Tokens
2. Create a new API token with upload permissions
3. In your GitHub repository, go to Settings → Secrets and variables → Actions
4. Add a new secret named `PYPI_API_TOKEN` with your PyPI token

### 2. Configure NPM Token

1. Go to [npmjs.com](https://npmjs.com) → Access Tokens
2. Generate a new **Classic Token** with **Automation** type
3. In your GitHub repository, go to Settings → Secrets and variables → Actions
4. Add a new secret named `NPM_TOKEN` with your npm token

### 3. Create Your First Release

1. Update the version in `openapi.json` if needed
2. Go to GitHub → Releases → Create a new release
3. Create a new tag following semantic versioning (e.g., `v1.0.0`)
4. Publish the release
5. The workflow will automatically build and publish both SDKs

## Usage

### Python

```bash
pip install sentienceapi-py
```

```python
from sentience_sdk import SentienceApiClient, SentienceConfiguration, SentienceObservationApi

# Configure API client
config = SentienceConfiguration(
    host="https://api.sentienceapi.com"
)
config.api_key['Authorization'] = "Bearer sk_live_your_api_key"

# Create client
with SentienceApiClient(config) as api_client:
    api = SentienceObservationApi(api_client)
    
    # Call the observe endpoint
    response = api.observe(
        observe_request={
            "url": "https://example.com",
            "mode": "reader"
        }
    )
    print(response)
```

### TypeScript/JavaScript

```bash
npm install @rcholic/sentience-sdk
```

```typescript
import { SentienceConfiguration, SentienceObservationApi } from '@rcholic/sentience-sdk';

// Configure API client
const config = new SentienceConfiguration({
  apiKey: 'Bearer sk_live_your_api_key',
  basePath: 'https://api.sentienceapi.com'
});

// Create client
const api = new SentienceObservationApi(config);

// Call the observe endpoint
const response = await api.observe({
  url: 'https://example.com',
  mode: 'reader'
});
console.log(response.data);
```

## Updating the SDK

1. Make changes to `openapi.json`
2. Commit and push your changes
3. Create a new release with an incremented version tag (e.g., `v1.0.1`)
4. The SDKs will be automatically regenerated and published

## Local Development

To test SDK generation locally before releasing:

```bash
# Install OpenAPI Generator
npm install @openapitools/openapi-generator-cli -g

# Generate Python SDK
openapi-generator-cli generate -i openapi.json -g python -o ./sdks/python --additional-properties=packageName=sentience_sdk

# Generate TypeScript SDK
openapi-generator-cli generate -i openapi.json -g typescript-axios -o ./sdks/typescript --additional-properties=npmName=@rcholic/sentience-sdk
```

## Notes

- The repository remains **private** while the published packages are **public**
- Never commit API keys or secrets to `openapi.json`
- The `openapi.json` should only reference public API endpoints
- SDKs are generated in the `sdks/` directory (gitignored to avoid committing generated code)
