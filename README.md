# HashiCorp Boundary Cancel Sessions Action

Cancel active sessions in HashiCorp Boundary to immediately terminate user access.

## Overview

This SGNL action integrates with HashiCorp Boundary to cancel active sessions. When executed, the specified session will be immediately terminated, disconnecting the user and revoking their access.

## Prerequisites

- HashiCorp Boundary instance
- Basic authentication credentials (username and password)
- Boundary API access
- Session ID to cancel
- Auth method ID for authentication

## Configuration

### Required Secrets

| Secret | Description |
|--------|-------------|
| `BASIC_USERNAME` | Username for HashiCorp Boundary authentication |
| `BASIC_PASSWORD` | Password for HashiCorp Boundary authentication |

### Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `ADDRESS` | HashiCorp Boundary API base URL | `https://boundary.example.com` |

### Input Parameters

| Parameter | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| `sessionId` | string | Yes | The Boundary session ID to cancel | `s_1234567890` |
| `authMethodId` | string | Yes | The Boundary auth method ID for authentication | `ampw_1234567890` |

### Output Structure

| Field | Type | Description |
|-------|------|-------------|
| `sessionId` | string | The session ID that was processed |
| `authMethodId` | string | The auth method ID used for authentication |
| `sessionCancelled` | boolean | Whether the session was successfully cancelled |
| `cancelledAt` | datetime | When the cancellation completed (ISO 8601) |

## Usage Example

### Job Request

```json
{
  "id": "cancel-session-001",
  "type": "nodejs-22",
  "script": {
    "repository": "github.com/sgnl-actions/hashicorp-boundary-cancel-sessions",
    "version": "v1.0.0",
    "type": "nodejs"
  },
  "script_inputs": {
    "sessionId": "s_1234567890",
    "authMethodId": "ampw_1234567890"
  },
  "environment": {
    "ADDRESS": "https://boundary.example.com",
    "LOG_LEVEL": "info"
  }
}
```

### Successful Response

```json
{
  "sessionId": "s_1234567890",
  "authMethodId": "ampw_1234567890",
  "sessionCancelled": true,
  "cancelledAt": "2024-01-15T10:30:00Z"
}
```

## How It Works

The action performs the following steps:

1. **Authenticate**: Uses the provided auth method ID and credentials to obtain an authentication token from Boundary
2. **Get Session Details**: Retrieves the current session information including its version number (required for updates)
3. **Cancel Session**: Cancels the specified session using the version number to ensure consistency

## Error Handling

The action includes comprehensive error handling with retryable and fatal error types:

### Retryable Errors (Framework will retry)
- **429 Rate Limit**: Boundary API rate limit exceeded
- **5xx Server Errors**: Boundary API server errors

### Fatal Errors (Will not retry)
- **401 Unauthorized**: Invalid username or password
- **403 Forbidden**: Insufficient permissions
- **404 Not Found**: Session not found
- **409 Conflict**: Session may already be cancelled or version mismatch
- **Missing Parameters**: Invalid or missing required parameters

## Development

### Local Testing

```bash
# Install dependencies
npm install

# Run tests
npm test

# Test locally with mock data
npm run dev

# Build for production
npm run build
```

### Running Tests

The action includes comprehensive unit tests covering:
- Input validation (sessionId, authMethodId)
- Secret validation (BASIC_USERNAME, BASIC_PASSWORD)
- Environment variable validation (ADDRESS)
- Empty parameter validation

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Check test coverage
npm run test:coverage
```

## Security Considerations

- **Credential Protection**: Never log or expose authentication credentials
- **Token Management**: Authentication tokens are ephemeral and obtained per-request
- **Audit Logging**: All operations are logged with timestamps
- **Input Validation**: All parameters are validated before API calls
- **Rate Limiting**: Includes delays between operations to avoid rate limits

## HashiCorp Boundary API Reference

This action uses the following HashiCorp Boundary API endpoints:
- [Authenticate](https://developer.hashicorp.com/boundary/api-docs/authmethods#authenticate)
- [Read Session](https://developer.hashicorp.com/boundary/api-docs/sessions#read)
- [Cancel Session](https://developer.hashicorp.com/boundary/api-docs/sessions#cancel)

## Troubleshooting

### Common Issues

1. **"Invalid or missing sessionId parameter"**
   - Ensure the `sessionId` parameter is provided and is a non-empty string
   - Verify the session ID exists in your Boundary instance

2. **"Invalid or missing authMethodId parameter"**
   - Ensure the `authMethodId` parameter is provided and is a non-empty string
   - Verify the auth method ID is correct for your Boundary instance

3. **"Missing required secrets: BASIC_USERNAME and BASIC_PASSWORD"**
   - Ensure both `BASIC_USERNAME` and `BASIC_PASSWORD` secrets are configured
   - Verify the credentials have the correct permissions

4. **"No URL specified. Provide address parameter or ADDRESS environment variable"**
   - Ensure the `ADDRESS` environment variable is set to your Boundary API URL
   - Example: `https://boundary.example.com`

5. **Authentication Errors (401)**
   - Verify your username and password are correct
   - Check that the auth method ID is valid

6. **Session Not Found (404)**
   - Verify the session ID is correct
   - Check that the session exists in Boundary

7. **Conflict Error (409)**
   - The session may already be cancelled
   - There may be a version mismatch - the action will handle retries automatically

## Version History

### v1.0.0
- Initial release
- Support for cancelling sessions via HashiCorp Boundary API
- Basic authentication support
- Integration with @sgnl-actions/utils package
- Comprehensive error handling with retryable/fatal error types

## License

MIT

## Support

For issues or questions, please contact SGNL Engineering or create an issue in this repository.
