# Google Drive MCP Server

A powerful Model Context Protocol (MCP) server that provides seamless integration with Google Drive, allowing AI models to search, list, and read files from Google Drive.

## 🚀 Features

### Tools

#### 1. `gdrive_search`
Search for files in your Google Drive with powerful full-text search capabilities.
- **Input**: 
  ```json
  {
    "query": "string (your search query)"
  }
  ```
- **Output**: List of files with:
  - File name
  - MIME type
  - File ID
  - Last modified time
  - File size

#### 2. `gdrive_read_file`
Read file contents directly using a Google Drive file ID.
- **Input**:
  ```json
  {
    "file_id": "string (Google Drive file ID)"
  }
  ```
- **Output**: File contents with appropriate format conversion

### Automatic File Format Handling

The server intelligently handles different Google Workspace file types:
- 📝 Google Docs → Markdown
- 📊 Google Sheets → CSV
- 📊 Google Presentations → Plain text
- 🎨 Google Drawings → PNG
- 📄 Text/JSON files → UTF-8 text
- 📦 Other files → Base64 encoded

## 🛠️ Getting Started

### Prerequisites
- Node.js (v16 or higher)
- npm or yarn
- A Google Cloud Project
- A Google Workspace or personal Google account

### Detailed Google Cloud Setup

1. **Create a Google Cloud Project**
   - Visit the [Google Cloud Console](https://console.cloud.google.com/projectcreate)
   - Click "New Project"
   - Enter a project name (e.g., "MCP GDrive Server")
   - Click "Create"
   - Wait for the project to be created and select it

2. **Enable the Google Drive API**
   - Go to the [API Library](https://console.cloud.google.com/apis/library)
   - Search for "Google Drive API"
   - Click on "Google Drive API"
   - Click "Enable"
   - Wait for the API to be enabled

3. **Configure OAuth Consent Screen**
   - Navigate to [OAuth consent screen](https://console.cloud.google.com/apis/credentials/consent)
   - Select User Type:
     - "Internal" if you're using Google Workspace
     - "External" for personal Google accounts
   - Click "Create"
   - Fill in the required fields:
     - App name: "MCP GDrive Server"
     - User support email: your email
     - Developer contact email: your email
   - Click "Save and Continue"
   - On the "Scopes" page:
     - Click "Add or Remove Scopes"
     - Add `https://www.googleapis.com/auth/drive.readonly`
     - Click "Update"
   - Click "Save and Continue"
   - Review the summary and click "Back to Dashboard"

4. **Create OAuth Client ID**
   - Go to [Credentials](https://console.cloud.google.com/apis/credentials)
   - Click "Create Credentials" at the top
   - Select "OAuth client ID"
   - Choose Application type: "Desktop app"
   - Name: "MCP GDrive Server Desktop Client"
   - Click "Create"
   - In the popup:
     - Click "Download JSON"
     - Save the file
     - Click "OK"

5. **Set Up Credentials in Project**
   ```bash
   # Create credentials directory
   mkdir credentials
   
   # Move and rename the downloaded JSON file
   mv path/to/downloaded/client_secret_*.json credentials/gcp-oauth.keys.json
   ```

### Installation

```bash
# Clone the repository
git clone https://github.com/felores/gdrive-mcp-server.git
cd gdrive-mcp-server

# Install dependencies
npm install

# Build the project
npm run build
```

### Authentication

1. Create a credentials directory and place your OAuth keys:
   ```bash
   mkdir credentials
   # Move your downloaded OAuth JSON file to the credentials directory as gcp-oauth.keys.json
   ```

2. Run the authentication command:
   ```bash
   node dist/index.js auth
   ```

3. Complete the OAuth flow in your browser
4. Credentials will be saved in `credentials/.gdrive-server-credentials.json`

## 🔧 Usage

### Output Format

The server supports two output formats controlled by the `OUTPUT_FORMAT` environment variable:

- **`text`** (default): Returns human-readable text responses
- **`json`**: Returns structured JSON responses for programmatic consumption

When `OUTPUT_FORMAT=json`:
- `gdrive_search` returns a JSON object with a `files` array containing file objects with `id`, `name`, `mimeType`, `modifiedTime`, and `size` fields
- `gdrive_read_file` returns a JSON object with `fileId`, `mimeType`, and `content` fields

Example JSON output from `gdrive_search`:
```json
{
  "files": [
    {
      "id": "mock_spreadsheet_123",
      "name": "Rent Value Update Report",
      "mimeType": "application/vnd.google-apps.spreadsheet",
      "modifiedTime": "2024-01-01T00:00:00.000Z",
      "size": "1024"
    }
  ],
  "total": 1
}
```

### As a Command Line Tool

```bash
# Start the server with default text output
node dist/index.js

# Start the server with JSON output
OUTPUT_FORMAT=json node dist/index.js
```

### Integration with Desktop App

Add this configuration to your app's server settings:

```json
{
  "mcpServers": {
    "gdrive": {
      "command": "node",
      "args": ["path/to/gdrive-mcp-server/dist/index.js"],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "path/to/gdrive-mcp-server/credentials/gcp-oauth.keys.json",
        "MCP_GDRIVE_CREDENTIALS": "path/to/gdrive-mcp-server/credentials/.gdrive-server-credentials.json",
        "OUTPUT_FORMAT": "json"
      }
    }
  }
}
```

Replace `path/to/gdrive-mcp-server` with the actual path to your installation directory.

### Alternative: Base64 Encoded Credentials (Containerized Environments)

For containerized deployments (Docker, Kubernetes) where managing files is difficult, you can provide credentials as base64 encoded environment variables:

```json
{
  "mcpServers": {
    "gdrive": {
      "command": "node",
      "args": ["path/to/gdrive-mcp-server/dist/index.js"],
      "env": {
        "OAUTH_CREDENTIALS_JSON_B64": "ewogICJpbnN0YWxsZWQiOiB7CiAgICAiY2xpZW50X2lkIjogIi4uLiIsCiAgICAiY2xpZW50X3NlY3JldCI6ICIuLi4iCiAgfQp9",
        "GDRIVE_CREDENTIALS_JSON_B64": "ewogICJhY2Nlc3NfdG9rZW4iOiAiLi4uIiwKICAicmVmcmVzaF90b2tlbiI6ICIuLi4iCn0=",
        "OUTPUT_FORMAT": "json"
      }
    }
  }
}
```

**To generate base64 encoded credentials:**

```bash
# Encode OAuth client credentials
base64 < credentials/gcp-oauth.keys.json | tr -d '\n'

# Encode user credentials
base64 < credentials/.gdrive-server-credentials.json | tr -d '\n'
```

**Environment Variable Priority:**

The server checks for credentials in this order:
1. `OAUTH_CREDENTIALS_JSON_B64` (base64 encoded OAuth client credentials)
2. `GOOGLE_APPLICATION_CREDENTIALS` (path to OAuth client credentials file)
3. Default path: `credentials/gcp-oauth.keys.json`

For user credentials:
1. `GDRIVE_CREDENTIALS_JSON_B64` (base64 encoded user OAuth tokens)
2. `MCP_GDRIVE_CREDENTIALS` (path to user credentials file)
3. Default path: `credentials/.gdrive-server-credentials.json`

### Example Usage

1. **Search for files**:
   ```typescript
   // Search for documents containing "quarterly report"
   const result = await gdrive_search({ query: "quarterly report" });
   ```

2. **Read file contents**:
   ```typescript
   // Read a specific file using its ID
   const contents = await gdrive_read_file({ file_id: "your-file-id" });
   ```

## 🔒 Security

- All sensitive credentials are stored in the `credentials` directory
- OAuth credentials and tokens are excluded from version control
- Read-only access to Google Drive
- Secure OAuth 2.0 authentication flow

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📝 License

This MCP server is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## 🔍 Troubleshooting

If you encounter issues:
1. Verify your Google Cloud Project setup
2. Ensure all required OAuth scopes are enabled
3. Check that credentials are properly placed in the `credentials` directory
4. Verify file permissions and access rights in Google Drive

## 📚 Additional Resources

- [Google Drive API Documentation](https://developers.google.com/drive/api/v3/reference)
- [OAuth 2.0 for Desktop Apps](https://developers.google.com/identity/protocols/oauth2/native-app)
- [Model Context Protocol Documentation](https://modelcontextprotocol.io)
