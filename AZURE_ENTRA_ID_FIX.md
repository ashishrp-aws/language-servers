# Azure Entra ID OAuth Support Fix

## Problem
Customer reported error when authenticating with Azure Entra ID:
```
AADSTS700016: Application with identifier 'dc-ntmmp1xkvnithy5axnpdxwpc5' was not found in the directory 'TIAA'
```

**Root Cause**: Azure Entra ID doesn't support Dynamic Client Registration (DCR). The code was attempting DCR and generating a random client ID that doesn't exist in the customer's Azure tenant.

## Solution
Allow users to provide a pre-registered Azure App client ID via the existing `headers` field in their MCP server configuration.

## Changes Made

### 1. Updated `OAuthClient.getValidAccessToken`
**File**: `server/aws-lsp-codewhisperer/src/language-server/agenticChat/tools/mcp/mcpOauthClient.ts`

- Added `clientId` parameter to options
- Pass `clientId` to `obtainClient` method

### 2. Updated `OAuthClient.obtainClient`
**File**: `server/aws-lsp-codewhisperer/src/language-server/agenticChat/tools/mcp/mcpOauthClient.ts`

- Added `preConfiguredClientId` parameter
- If provided, skip DCR and use the pre-configured client ID
- Updated error message to guide users on configuration

### 3. Updated `McpManager.initOneServer`
**File**: `server/aws-lsp-codewhisperer/src/language-server/agenticChat/tools/mcp/mcpManager.ts`

- Extract `X-OAuth-Client-Id` header from `cfg.headers`
- Pass extracted `clientId` to `OAuthClient.getValidAccessToken`

## Customer Configuration

For Azure Entra ID (or any OAuth provider without DCR support), users should:

### 1. Register an App in Azure Entra ID
- Go to Azure Portal → Azure Active Directory → App registrations
- Create a new registration
- Add redirect URI: `http://127.0.0.1` (with any port, e.g., `http://127.0.0.1:41877`)
- Note the Application (client) ID

### 2. Update MCP Server Configuration
Add the `X-OAuth-Client-Id` header to the MCP server configuration in `~/.aws/amazonq/agents/default.json`:

```json
{
  "mcpServers": {
    "my-azure-server": {
      "url": "https://your-mcp-server.com",
      "headers": {
        "X-OAuth-Client-Id": "YOUR_AZURE_APP_CLIENT_ID"
      }
    }
  }
}
```

## Error Messages

### Before Fix
```
OAuth: AS does not support dynamic registration
```

### After Fix
```
OAuth: AS does not support dynamic registration. For Azure Entra ID, add "headers": {"X-OAuth-Client-Id": "YOUR_AZURE_APP_CLIENT_ID"} to your MCP server configuration.
```

## Testing

1. Configure an MCP server with Azure Entra ID OAuth
2. Add the `X-OAuth-Client-Id` header with a valid Azure App client ID
3. Verify authentication succeeds without DCR errors
4. Verify the client ID is cached and reused on subsequent authentications

## Benefits of Headers Approach

- Reuses existing `headers` field infrastructure
- No new config schema changes needed
- Consistent with how other metadata is passed (e.g., Authorization headers)
- The `X-OAuth-Client-Id` header is only used internally for OAuth setup, not sent to the MCP server

## Related Ticket
- Ticket: V1947019868
- Customer: Presidio Networked Solutions Group, LLC (Account: 054037099556)
