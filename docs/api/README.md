# API Reference

This document provides comprehensive documentation for all API endpoints in the Payment Copilot application.

## Base URL

All API endpoints are prefixed with `/api/` and run on the same domain as your application.

## Authentication

Most endpoints require authentication. Include the user's Firebase ID token in the Authorization header:

```
Authorization: Bearer <firebase_id_token>
```

## Endpoints

### OpenAI

#### POST `/api/openai/chat`

Streams a chat completion from OpenAI.

**Request Body:**
```json
{
  "messages": [
    {
      "role": "user",
      "content": "Your message here"
    }
  ]
}
```

**Response:** Stream of text chunks

**Example:**
```bash
curl -X POST http://localhost:3000/api/openai/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{"messages": [{"role": "user", "content": "Hello"}]}'
```

#### POST `/api/openai/transcribe`

Transcribes audio using OpenAI's Whisper model.

**Request:** Multipart form data with audio file

**Response:**
```json
{
  "text": "Transcribed text"
}
```

### Anthropic

#### POST `/api/anthropic/chat`

Streams a chat completion from Anthropic's Claude.

**Request Body:**
```json
{
  "messages": [
    {
      "role": "user",
      "content": "Your message here"
    }
  ]
}
```

**Response:** Stream of text chunks

### Replicate

#### POST `/api/replicate/generate-image`

Generates an image using Stable Diffusion via Replicate.

**Request Body:**
```json
{
  "prompt": "A beautiful landscape",
  "width": 512,
  "height": 512
}
```

**Response:**
```json
{
  "imageUrl": "https://replicate.delivery/..."
}
```

### Deepgram

#### GET `/api/deepgram`

Returns the Deepgram API key for client-side usage.

**Response:**
```json
{
  "apiKey": "your_deepgram_api_key"
}
```

## Error Handling

All endpoints return appropriate HTTP status codes:

- `200` - Success
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `500` - Internal Server Error

Error responses include a JSON object with error details:

```json
{
  "error": "Error message",
  "code": "ERROR_CODE"
}
```

## Rate Limiting

API endpoints may be rate limited. Check response headers for rate limit information:

- `X-RateLimit-Limit` - Requests per time window
- `X-RateLimit-Remaining` - Remaining requests
- `X-RateLimit-Reset` - Time when limit resets

## Webhooks

Currently, no webhook endpoints are implemented. This section will be updated when webhooks are added.

## SDKs and Libraries

### JavaScript/TypeScript

```typescript
// Example usage with fetch
const response = await fetch('/api/openai/chat', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${firebaseToken}`
  },
  body: JSON.stringify({
    messages: [{ role: 'user', content: 'Hello' }]
  })
});

const reader = response.body?.getReader();
// Handle streaming response
```

### Python

```python
import requests

response = requests.post(
    'http://localhost:3000/api/openai/chat',
    headers={
        'Content-Type': 'application/json',
        'Authorization': f'Bearer {firebase_token}'
    },
    json={
        'messages': [{'role': 'user', 'content': 'Hello'}]
    }
)
```

## Testing

Use the provided Postman collection or curl commands to test the API endpoints. Ensure you have valid authentication tokens before testing.

## Support

For API support, please:
1. Check the [Troubleshooting Guide](../getting-started.md#troubleshooting)
2. Review error messages in the browser console
3. Verify your API keys are correctly configured
