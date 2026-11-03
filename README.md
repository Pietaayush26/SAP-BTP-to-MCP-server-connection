SAP BTP Administration MCP Server + GitHub Copilot
Connect SAP Business Technology Platform (BTP) Administration MCP Server with GitHub Copilot in Visual Studio Code.

!SAP BTP !MCP !GitHub Copilot !VS Code

1. Introduction

The Model Context Protocol (MCP) allows AI assistants to communicate with external systems through a standardized interface.

SAP provides an MCP Server for SAP BTP Administration that allows compatible AI clients, including GitHub Copilot in Visual Studio Code, to discover and invoke SAP BTP administrative tools.

Instead of manually navigating the SAP BTP Cockpit or executing repetitive CLI commands, administrators can interact with their SAP BTP landscape using natural language.

What are we building?

We are connecting:

SAP BTP Administration MCP Server: Exposes SAP BTP administrative capabilities.
Visual Studio Code: Development environment hosting the MCP client.
GitHub Copilot: AI assistant that discovers and uses MCP tools.
SAP Authentication: Provides authorized access to the SAP BTP landscape.
Architecture
text
             Developer
                 |
                 |
          Natural Language
                 |
                 v
       +--------------------+
       |  GitHub Copilot    |
       |     VS Code        |
       +--------------------+
                 |
                 | MCP Protocol
                 | Streamable HTTP
                 v
       +--------------------+
       |   SAP BTP MCP      |
       |   Administration   |
       |      Server        |
       +--------------------+
                 |
                 | SAP Authentication
                 | OAuth / SSO
                 v
       +--------------------+
       |    SAP BTP         |
       | Global Account     |
       | Subaccounts        |
       | Security           |
       | Services           |
       | Connectivity       |
       +--------------------+

2. Prerequisites

Before starting, make sure you have:

Requirement	Description
Visual Studio Code	Latest supported VS Code version
GitHub Copilot	Copilot extension enabled
GitHub Account	Account with Copilot access
SAP BTP Account	Access to an SAP BTP global account
SAP Identity	SAP user with appropriate administrative permissions
MCP Support	VS Code version supporting MCP servers

Install or enable the following VS Code extensions:

GitHub Copilot
GitHub Copilot Chat
3. SAP BTP MCP Server Details

SAP provides a dedicated MCP endpoint for BTP Administration.

Available endpoints
Authentication	Endpoint
Single Sign-On (SSO)	https://sso.mcp.btp.cloud.sap/mcp
Direct Connection	https://proxy.c-769d49e.kyma.ondemand.com/mcp

For this tutorial, we will use the SSO-based connection, which is the recommended starting point.

Supported administration domains
Domain	Capabilities
Accounts	Manage global accounts, directories and subaccounts
Security	Manage users, roles, role collections and trust
Services	Manage entitlements, quotas and subscriptions
Connectivity	Manage destinations and connectivity configurations

The MCP server dynamically exposes its available tools to the AI client. Individual tools do not need to be manually registered.

4. Configure MCP in VS Code
Step 1: Open your project

Open your application or workspace in Visual Studio Code.

Step 2: Create the MCP configuration

Inside your project, create the following folder and file:

text
.vscode/
    mcp.json


Alternatively:

Open the Command Palette using Ctrl + Shift + P.
Search for MCP: Open User Configuration or use the workspace MCP configuration.
Create or open the appropriate MCP configuration file.
Step 3: Configure the SAP BTP MCP server

Add the following configuration to .vscode/mcp.json using the configuration format from the SAP Community guide:

json
{
  "mcpServers": {
    "BTP Administration": {
      "type": "streamableHttp",
      "url": "https://sso.mcp.btp.cloud.sap/mcp"
    }
  }
}


Save the file.

Note: VS Code MCP configuration schemas can vary by version. If your VS Code installation expects the newer servers root property or an explicit OAuth configuration, use the schema accepted by your installed VS Code version. See the troubleshooting section below.
Step 4: Configure OAuth Client ID

When VS Code requests the OAuth Client ID, enter:

text
e789ba01-5612-47ee-bfe7-79e26411c1ca


This is the client ID published in the SAP Community guide.

VS Code stores the OAuth client configuration, so it should not need to request it again for subsequent connections.

Step 5: Authenticate with SAP
Start the MCP connection from VS Code.
When prompted, open the browser authentication window.
Sign in using your SAP identity.
Complete the OAuth/SSO authorization flow.
Return to Visual Studio Code.

After successful authentication, the SAP BTP Administration MCP server should become available to GitHub Copilot.

5. Start and Verify the MCP Connection
Step 1: Open GitHub Copilot Chat

Open Copilot Chat from the VS Code sidebar.

Step 2: Select Agent Mode

Switch the chat interface to Agent Mode.

Agent Mode allows Copilot to discover and invoke available MCP tools.

Step 3: Check MCP server status

Open the MCP server management interface in VS Code.

You should see an entry similar to:

text
BTP Administration

Status: Connected
Transport: Streamable HTTP
Authentication: OAuth / SSO


The exact status labels depend on your VS Code version.

Step 4: Verify available tools

Inspect the tools exposed by the BTP Administration MCP server.

The available tools depend on SAP-side support, your permissions and the operations exposed by the service.

6. Test with GitHub Copilot

Once the connection is active, you can interact with SAP BTP using natural language.

Example 1: Global Account
text
Show me the global accounts associated
with my SAP identity.

Example 2: Subaccounts
text
List all subaccounts available in my
SAP BTP global account.

Example 3: Regions
text
Show the available SAP BTP regions.

Example 4: Service Entitlements
text
Show the service entitlement capacity
for my global account.

Example 5: User Management
text
List users assigned to my SAP BTP
subaccount.

Example 6: Role Collections
text
Show all available role collections
in my production subaccount.

Example 7: Create a Subaccount
text
Create a development subaccount
in the EU10 region.

First show me the configuration
and ask for confirmation before
creating the subaccount.


Important: Administrative operations can modify your SAP environment. Review the target, scope and consequences before approving any write or destructive operation.

7. Alternative: Direct Connection

If you cannot use SSO, SAP also provides a direct connection endpoint.

This approach requires:

SAP username and password.
Base64-encoded credentials.
IAS tenant subdomain.
Step 1: Encode your credentials

The expected format is:

text
username:password


macOS / Linux:

bash
echo -n "<username>:<password>" | base64


Windows PowerShell:

powershell
[Convert]::ToBase64String(
  [Text.Encoding]::UTF8.GetBytes(
    '<username>:<password>'
  )
)

Step 2: Configure VS Code

Create or update .vscode/mcp.json:

json
{
  "inputs": [
    {
      "id": "btp-credentials",
      "type": "promptString",
      "description": "Base64-encoded BTP credentials",
      "password": true
    },
    {
      "id": "btp-origin",
      "type": "promptString",
      "description": "IAS tenant subdomain"
    }
  ],
  "mcpServers": {
    "BTP Administration": {
      "type": "streamableHttp",
      "url": "https://proxy.c-769d49e.kyma.ondemand.com/mcp",
      "headers": {
        "Authorization": "Basic ${input:btp-credentials}",
        "X-Platform-Origin": "${input:btp-origin}"
      }
    }
  }
}


VS Code prompts for the credentials when establishing the connection. According to the SAP guide, these prompted values are not stored on disk.

Security recommendation: Prefer SSO wherever possible. Base64 is encoding, not encryption. Never commit credentials, access tokens or sensitive configuration values to Git.

8. Troubleshooting
Issue 1: MCP server is not connecting

Check:

Your internet connection.
The MCP endpoint URL.
Whether your VS Code version supports remote MCP servers.
Whether the authentication flow completed successfully.

Try restarting the MCP server from VS Code.

Issue 2: OAuth authentication problems

Check the OAuth client configuration.

The SAP-provided client ID is:

text
e789ba01-5612-47ee-bfe7-79e26411c1ca


Some VS Code versions or OAuth redirect configurations may require an explicit OAuth client configuration.

For example, the newer servers configuration style can look like:

json
{
  "servers": {
    "BTP Administration": {
      "type": "http",
      "url": "https://sso.mcp.btp.cloud.sap/mcp",
      "oauth": {
        "clientId": "e789ba01-5612-47ee-bfe7-79e26411c1ca"
      }
    }
  }
}


Use this only if it matches the schema supported by your installed VS Code version.

Issue 3: Only one MCP tool is visible

A successful connection does not necessarily mean every administrative operation is available.

Check:

The tools listed by the MCP server.
Your SAP BTP user permissions.
The capabilities supported by your account and tenant.
Whether the issue is related to SAP-side tool availability.

The SAP Community discussion includes reports of clients showing only Navigation-buildLink despite a successful connection. This should be investigated rather than assuming all tools are available.

Issue 4: Some operations fail

Some operations may require additional authentication or environment-specific configuration.

For example, Cloud Foundry-related operations may require a separate Cloud Foundry login:

bash
cf login -a <api.cf.<region>.hana.ondemand.com>


A successful MCP OAuth connection does not automatically authenticate every underlying SAP service or environment.

Issue 5: SAP BTP trial limitations

Some administrative capabilities may not be available for trial accounts.

Verify the operation against a supported account and check the permissions associated with your SAP identity.

SAP Support

If the problem persists, raise an SAP support request using:

text
Component: BC-CP-ADMINMCP

9. Security Considerations

The MCP server operates with the scope of your authenticated SAP user identity.

This is particularly important when one SAP identity is associated with multiple global accounts.

Follow these recommendations:

Use a dedicated SAP identity for testing where possible.
Avoid using a productive administrator identity for experiments.
Review OAuth authorization and granted permissions.
Do not expose credentials in source code.
Avoid automatic approval of sensitive administrative operations.
Review Copilot's proposed actions before executing them.
Test write operations in a non-production environment first.
10. Final Outcome

After completing this integration, your development environment will have the following architecture:

text
GitHub Copilot Chat
        |
        v
VS Code Agent Mode
        |
        v
BTP Administration MCP
        |
        v
SAP OAuth / SSO
        |
        v
SAP BTP Landscape


You can now use GitHub Copilot as a natural-language interface for supported SAP BTP administration operations.

Instead of manually navigating different SAP BTP Cockpit screens, you can ask questions, inspect account information and perform supported administrative tasks through MCP.

11. References
Original SAP Community Guide:
https://community.sap.com/t5/technology-blog-posts-by-sap/step-by-step-guide-to-connecting-the-mcp-server-for-sap-btp-administration/ba-p/14474020
VS Code MCP Documentation:
https://code.visualstudio.com/docs/copilot/customization/mcp-servers
Model Context Protocol:
https://modelcontextprotocol.io/

Authoring note: This guide is based on the SAP Community article. Endpoint availability, authentication behavior, configuration schemas and supported tools should always be verified against the current SAP and VS Code documentation.
