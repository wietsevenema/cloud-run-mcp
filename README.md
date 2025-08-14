# Cloud Run MCP Server

**Deploy apps to Google Cloud Run with your AI agent.**

This repository contains a Model Context Protocol (MCP) server that enables AI agents and IDEs to deploy applications to Google Cloud Run.

<img src="https://github.com/user-attachments/assets/9fdcec30-2b38-4362-9eb1-54cab09e99d4" width="800">

> [!NOTE]  
> This is the repository of an MCP server to deploy code to Cloud Run, to learn how to **host** MCP servers on Cloud Run, [visit the Cloud Run documentation](https://cloud.google.com/run/docs/host-mcp-servers).

## Features

*   Deploy web services to Cloud Run.
*   List and manage Cloud Run services.
*   View service logs and error messages.
*   Create new Google Cloud projects.
*   Uses local `gcloud` credentials for authentication.
*   Can be deployed as a remote MCP service.

---

## Getting Started

### Prerequisites

*   An AI agent or IDE that supports the Model Context Protocol (MCP).
*   [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) installed and authenticated.

### Installation

You can use this MCP server in three ways: as a local server, as a remote server on Cloud Run, or as a Gemini CLI extension.

---

## Usage

### Local MCP Server

Run the Cloud Run MCP server on your local machine using your local Google Cloud credentials. This is the recommended approach for use with AI-assisted IDEs such as Cursor or desktop AI applications such as Gemini CLI or Claude Code.

**1. Authenticate with Google Cloud:**

```bash
gcloud auth login --update-adc
```

**2. Configure your MCP Client:**

Update the MCP configuration file of your client to use one of the following methods:

#### Using Node.js

*   Requires [Node.js](https://nodejs.org/en/download/) (LTS version recommended).

```json
   "cloud-run": {
     "command": "npx",
     "args": ["-y", "https://github.com/GoogleCloudPlatform/cloud-run-mcp"]
   }
```

#### Using Docker

*   Requires [Docker](https://www.docker.com/get-started/).

See Docker's [MCP catalog](https://hub.docker.com/mcp/server/cloud-run-mcp/overview), or use these manual instructions:

```json
   "cloud-run": {
     "command": "docker",
     "args": [
       "run",
       "-i",
       "--rm",
       "-e",
       "GOOGLE_APPLICATION_CREDENTIALS",
       "-v",
       "/local-directory:/local-directory",
       "mcp/cloud-run-mcp:latest"
     ],
     "env": {
       "GOOGLE_APPLICATION_CREDENTIALS": "/Users/slim/.config/gcloud/application_default-credentials.json"
     }
   }
```

### Remote MCP Server

Run the Cloud Run MCP server on Cloud Run itself and connect to it securely from your local machine. With this option, you can only deploy code to the same Google Cloud project where the MCP server is running.

> [!WARNING]
> Do not use the remote MCP server without authentication. The following instructions use IAM authentication to secure the connection.

**1. Deploy the MCP Server to Cloud Run:**

```bash
gcloud config set project YOUR_PROJECT_ID
gcloud run deploy cloud-run-mcp --image us-docker.pkg.dev/cloudrun/container/mcp --no-allow-unauthenticated
```
When prompted, pick a region (e.g., `europe-west1`).

**2. Run a Local Proxy:**

```bash
gcloud run services proxy cloud-run-mcp --port=3000 --region=REGION --project=PROJECT_ID
```
This creates a local proxy on port 3000 that forwards requests to the remote MCP server.

**3. Configure your MCP Client:**

```json
   "cloud-run": {
     "url": "http://localhost:3000/sse"
   }
```

If your MCP client does not support the `url` attribute, you can use [mcp-remote](https://www.npmjs.com/package/mcp-remote):

```json
   "cloud-run": {
     "command": "npx",
     "args": ["-y", "mcp-remote", "http://localhost:3000/sse"]
   }
```

### Gemini CLI Extension

To install this as a [Gemini CLI](https://github.com/google-gemini/gemini-cli) extension, run the following command:

```bash
mkdir -p ~/.gemini/extensions/cloud-run/gemini-extension && \
  curl -s -L https://raw.githubusercontent.com/GoogleCloudPlatform/cloud-run-mcp/main/gemini-extension.json > ~/.gemini/extensions/cloud-run/gemini-extension.json && \
  curl -s -L https://raw.githubusercontent.com/GoogleCloudPlatform/cloud-run-mcp/main/gemini-extension/Gemini.md > ~/.gemini/extensions/cloud-run/gemini-extension/Gemini.md
```

---

## Reference

### Tools

*   `deploy-file-contents`: Deploys files to Cloud Run by providing their contents directly.
*   `list-services`: Lists Cloud Run services in a given project and region.
*   `get-service`: Gets details for a specific Cloud Run service.
*   `get-service-log`: Gets Logs and Error Messages for a specific Cloud Run service.
*   `deploy-local-files`*: Deploys files from the local file system to a Google Cloud Run service.
*   `deploy-local-folder`*: Deploys a local folder to a Google Cloud Run service.
*   `list-projects`*: Lists available GCP projects.
*   `create-project`*: Creates a new GCP project and attach it to the first available billing account. A project ID can be optionally specified.

*   only available when running locally*

### Prompts

Prompts are natural language commands that can be used to perform common tasks.

*   `deploy`: Deploys the current working directory to Cloud Run.
*   `logs`: Gets the logs for a Cloud Run service.

### Configuration

You can configure the MCP server with the following environment variables:

*   `GOOGLE_CLOUD_PROJECT`: Your Google Cloud project ID.
*   `GOOGLE_CLOUD_REGION`: The default region to use for deployments.
*   `DEFAULT_SERVICE_NAME`: The default service name to use for deployments.
*   `SKIP_IAM_CHECK`: Set to `true` to skip the IAM check on a remote server.

---

## Contributing

Contributions are welcome! Please see the [contributing guidelines](CONTRIBUTING.md) for more information.
