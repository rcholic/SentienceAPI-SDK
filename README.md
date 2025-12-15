# Sentience API SDK

Official Python and TypeScript/JavaScript SDKs for the Sentience API.

## Installation

### Python

```bash
pip install sentienceapi-py
```

### TypeScript/JavaScript

```bash
npm install @rcholic/sentience-sdk
```

## Usage

### Python

```python
from sentience_sdk import SentienceApiClient, SentienceConfiguration, SentienceObservationApi

# Configure API client with your API key
config = SentienceConfiguration(api_key="sk_live_your_api_key")

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
    
    # With options for map mode
    response_with_options = api.observe(
        observe_request={
            "url": "https://example.com",
            "mode": "map",
            "options": {
                "render_quality": "high",
                "limit": 100
            }
        }
    )
    print(response_with_options)
    
    # Visual mode example
    visual_response = api.observe(
        observe_request={
            "url": "https://example.com",
            "mode": "visual",
            "options": {
                "render_quality": "high",
                "limit": 50
            }
        }
    )
    print(visual_response)
```

### TypeScript/JavaScript

```typescript
import { SentienceConfiguration, SentienceObservationApi } from '@rcholic/sentience-sdk';

// Configure API client
const config = new SentienceConfiguration({
  apiKey: 'Bearer sk_live_your_api_key'
});

// Create client
const api = new SentienceObservationApi(config);

// Call the observe endpoint
const response = await api.observe({
  url: 'https://example.com',
  mode: 'reader'
});
console.log(response.data);

// With options for map mode
const responseWithOptions = await api.observe({
  url: 'https://example.com',
  mode: 'map',
  options: {
    render_quality: 'high',
    limit: 100
  }
});
console.log(responseWithOptions.data);

// Visual mode example
const visualResponse = await api.observe({
  url: 'https://example.com',
  mode: 'visual',
  options: {
    render_quality: 'high',
    limit: 50
  }
});
console.log(visualResponse.data);
```

## Support

For API documentation and support, visit [sentienceapi.com](https://sentienceapi.com) or contact support@sentienceapi.com.
