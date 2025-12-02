# Azure Entra ID OAuth Configuration for MCP Servers

This document explains how to configure MCP servers to work with Azure Entra ID OAuth authentication.

## Problem

Azure Entra ID has specific requirements that differ from standard OAuth implementations:

1. **No Dynamic Client Registration (DCR)**: Azure Entra ID doesn't support automatic client registration
2. **Pre-registered Redirect URIs**: All redirect URIs must be registered in advance in the Azure App registration
3. **No `resource` parameter**: Azure uses `scope` instead of the OAuth 2.0 `resource` parameter

## Solution

Use pre-registered Azure App credentials and redirect URIs in your MCP server configuration.

## Setup Steps

### 1. Register an Azure App

1. Go to [Azure Portal](https://portal.azure.com) → Azure Active Directory → App registrations
2. Click "New registration"
3. Configure:
   - **Name**: Choose a descriptive name (e.g., "MCP Server Access")
   - **Supported account types**: Choose appropriate option for your organization
   - **Redirect URI**: Add `http://127.0.0.1:8080` (or your preferred port)
4. Click "Register"
5. Note the **Application (client) ID** from the Overview page

### 2. Configure Redirect URIs

1. In your Azure App registration, go to **Authentication**
2. Under "Redirect URIs", add:
   - `http://127.0.0.1:8080` (or your chosen port)
   - Optionally add other ports like `http://127.0.0.1:8081`, `http://127.0.0.1:8082` for flexibility
3. Under "Advanced settings":
   - Enable "Allow public client flows" if needed
   - Configure other settings as required by your organization
4. Click "Save"

### 3. Configure MCP Server

Add the Azure App credentials to your MCP server configuration in `~/.aws/amazonq/agents/default.json`:

```json
{
  "mcpServers": {
    "my-azure-server": {
      "url": "https://your-mcp-server.example.com",
      "headers": {
        "X-OAuth-Client-Id": "your-azure-app-client-id-here",
        "X-OAuth-Redirect-URI": "http://127.0.0.1:8080"
      }
    }
  }
}
```

## Configuration Options

### Required Headers

- **`X-OAuth-Client-Id`**: Your Azure App's Application (client) ID
- **`X-OAuth-Redirect-URI`**: The redirect URI registered in your Azure App (must match exactly)

### Example Configurations

#### Basic Configuration
```json
{
  "mcpServers": {
    "azure-mcp": {
      "url": "https://api.example.com/mcp",
      "headers": {
        "X-OAuth-Client-Id": "12345678-1234-1234-1234-123456789012",
        "X-OAuth-Redirect-URI": "http://127.0.0.1:8080"
      }
    }
  }
}
```

#### With Custom Port
```json
{
  "mcpServers": {
    "azure-mcp": {
      "url": "https://api.example.com/mcp",
      "headers": {
        "X-OAuth-Client-Id": "12345678-1234-1234-1234-123456789012",
        "X-OAuth-Redirect-URI": "http://127.0.0.1:9000"
      }
    }
  }
}
```

## Important Notes

1. **Exact Match Required**: The `X-OAuth-Redirect-URI` must exactly match one of the redirect URIs registered in your Azure App
2. **Port Availability**: Ensure the port specified in the redirect URI is available when Amazon Q attempts OAuth authentication
3. **Firewall**: Make sure your firewall allows connections to the specified port
4. **Case Sensitivity**: Header names are case-insensitive, so `x-oauth-client-id` works the same as `X-OAuth-Client-Id`

## Troubleshooting

### Common Errors

#### `AADSTS50011: The redirect URI does not match`
- **Cause**: The redirect URI in your configuration doesn't match what's registered in Azure
- **Solution**: Verify the redirect URI in your Azure App registration matches exactly what you specified in `X-OAuth-Redirect-URI`

#### `AADSTS700016: Application with identifier 'xxx' was not found`
- **Cause**: The client ID is incorrect or the app isn't registered in the correct tenant
- **Solution**: Verify the `X-OAuth-Client-Id` matches your Azure App's Application (client) ID

#### `Failed to bind to custom redirect URI`
- **Cause**: The port specified in the redirect URI is already in use or blocked
- **Solution**: Choose a different port and update both your Azure App registration and MCP configuration

### Verification Steps

1. Check Azure App registration:
   - Verify Application (client) ID
   - Confirm redirect URI is registered
   - Ensure "Allow public client flows" is enabled if needed

2. Check MCP configuration:
   - Verify `X-OAuth-Client-Id` matches Azure App ID
   - Verify `X-OAuth-Redirect-URI` matches registered redirect URI exactly
   - Check for typos in header names

3. Check network:
   - Ensure the specified port is available
   - Verify firewall settings allow the port

## Support

If you continue to experience issues:

1. Check Amazon Q logs for detailed error messages
2. Verify your Azure App configuration
3. Test with a simple redirect URI like `http://127.0.0.1:8080`
4. Contact support with your configuration (redact sensitive information)