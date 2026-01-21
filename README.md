# Voyp Model Context Protocol server

[![smithery badge](https://smithery.ai/badge/@paulotaylor/voyp-mcp)](https://smithery.ai/server/@paulotaylor/voyp-mcp)

The Model Context Protocol (MCP) is an open standard that enables AI systems to interact seamlessly with various data sources and tools, facilitating secure, two-way connections.

Developed by Anthropic, the Model Context Protocol (MCP) enables AI assistants like Claude to seamlessly integrate with VOYP's calling capabilities. This integration provides AI models with possibility of making phone calls and monitor their progress.

The Voyp MCP server allows you to:
- Construct robust call contexts to use when making calls
- Search for business information when calling restaurants, dentists, etc...
- Call and make appointments, reservations, consultations, inquiries, etc...
- Provide status of the call
- Hangup call

<a href="https://glama.ai/mcp/servers/nlah6xt0ml"><img width="380" height="200" src="https://glama.ai/mcp/servers/nlah6xt0ml/badge" alt="Voyp Server MCP server" /></a>

## Prerequisites 🔧

Before you use Voyp, you need:

- [Voyp API key](https://voyp.app/app.html)
  - You will also need to buy credits to spend while making calls. You can also buy credits [here](https://voyp.app/app.html)
- [Claude Desktop](https://claude.ai/download), [Dive](https://github.com/OpenAgentPlatform/Dive) [Goose](https://github.com/block/goose) or other compatible clients
- [Node.js](https://nodejs.org/) (v20 or higher)
  - You can verify your Node.js installation by running:
    - `node --version`
- [Git](https://git-scm.com/downloads) installed (only needed if using Git installation method)
  - On macOS: `brew install git`
  - On Linux: 
    - Debian/Ubuntu: `sudo apt install git`
    - RedHat/CentOS: `sudo yum install git`
  - On Windows: Download [Git for Windows](https://git-scm.com/download/win)

## Remote Voyp MCP server (Http Streamable)

You can connect directly to the stream-enabled endpoint here:

ht<span>tps://</span>api.voyp.app/mcp/stream

Voyp supports the OAuth2 authentication flow 

You must add the Authorization header to your requests with your Voyp API Key or OAuth2 access tooken:

```
Authorization: Bearer sk_xyz
```

## Voyp MCP server installation for [Goose](https://github.com/block/goose) ⚡ (Stdio)

To install the voyp-mcp server you will need to add the extension manually.

![Adding VOYP MCP server to Goose](./assets/voyp-goose-extension.png)

Voyp + Goose Demo:

[![Voyp + Goose Demo](./assets/voyp-goose.gif)](https://www.youtube.com/watch?v=mZaLncjvYOc)


## Voyp MCP server installation for Claude Desktop ⚡ (Stdio)

To install the voyp-mcp server, you can use the following methods:
1. Installing via Smithery
2. Running with NPX 
3. Git installation

### 1. Installing via Smithery

To install Voyp Model Context Protocol server for Claude Desktop automatically via [Smithery](https://smithery.ai/server/@paulotaylor/voyp-mcp):

```bash
npx -y @smithery/cli install @paulotaylor/voyp-mcp --client claude
```

### 2. Running with NPX 

```bash
npx -y voyp-mcp@0.1.0   
```

Although you can launch a server on its own, it's not particularly helpful in isolation. Instead, you should integrate it into an MCP client. Below is an example of how to configure the Claude Desktop app to work with the voyp-mcp server.

### Configuring the Claude Desktop app ⚙️
### For macOS:

```bash
# Create the config file if it doesn't exist
touch "$HOME/Library/Application Support/Claude/claude_desktop_config.json"

# Opens the config file in TextEdit 
open -e "$HOME/Library/Application Support/Claude/claude_desktop_config.json"

# Alternative method using Visual Studio Code (requires VS Code to be installed)
code "$HOME/Library/Application Support/Claude/claude_desktop_config.json"
```

### For Windows:
```bash
code %APPDATA%\Claude\claude_desktop_config.json
```

### Add the Voyp server configuration:

Replace `your-VOYP-api-key` with your actual [VOYP API key](https://voyp.app/app.html).

```json
{
  "mcpServers": {
    "voyp-mcp": {
      "command": "npx",
      "args": ["-y", "voyp-mcp"],
      "env": {
        "VOYP_API_KEY": "your-VOYP-api-key"
      }
    }
  }
}
```

### 3. Git Installation

1. Clone the repository:
```bash
git clone https://github.com/paulotaylor/voyp-mcp.git
cd voyp-mcp
```

2. Install dependencies:
```bash
npm install
```

3. Build the project:
```bash
npm run build
```
### Configuring the Claude Desktop app ⚙️
Follow the configuration steps outlined in the [Configuring the Claude Desktop app](#configuring-the-claude-desktop-app-️) section above, using the below JSON configuration.

Replace `your-VOYP-api-key-here` with your actual [VOYP API key](https://voyp.app/app.html) and `/path/to/voyp-mcp` with the actual path where you cloned the repository on your system.

```json
{
  "mcpServers": {
    "voyp": {
      "command": "npx",
      "args": ["/path/to/voyp-mcp/build/index.js"],
      "env": {
        "VOYP_API_KEY": "your-VOYP-api-key"
      }
    }
  }
}
```

## Usage in Claude Desktop App 🎯

Once the installation is complete, and the Claude desktop app is configured, you must completely close and re-open the Claude desktop app to see the voyp-mcp server. You should see a hammer icon in the bottom left of the app, indicating available MCP tools, you can click on the hammer icon to see more details on the start_call and hangup_call tools.


Now claude will have complete access to the voyp-mcp server, including the start_call and hangup_call tools. 

Voyp + Claude Desktop Demo:

[![Voyp + Claude Desktop Demo](./assets/voyp-claude.gif)](https://www.youtube.com/watch?v=E0D1KqOBRuo)


## Troubleshooting 🛠️

### Common Issues

1. **Server Not Found**
   - Verify the npm installation by running `npm --verison`
   - Check Claude Desktop configuration syntax by running `code ~/Library/Application\ Support/Claude/claude_desktop_config.json`
   - Ensure Node.js is properly installed by running `node --version`
   
2. **NPX related issues**
  - If you encounter errors related to `npx`, you may need to use the full path to the npx executable instead. 
  - You can find this path by running `which npx` in your terminal, then replace the `"command":  "npx"` line with `"command": "/full/path/to/npx"` in your configuration.

3. **API Key Issues**
   - Confirm your VOYP API key is valid
   - Check the API key is correctly set in the config
   - Verify no spaces or quotes around the API key

## Acknowledgments ✨

- [Model Context Protocol](https://modelcontextprotocol.io) for the MCP specification
- [Anthropic](https://anthropic.com) for Claude Desktop
