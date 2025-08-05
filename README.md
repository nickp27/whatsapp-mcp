# WhatsApp MCP Server

This is a Model Context Protocol (MCP) server for WhatsApp.

With this you can search and read your personal Whatsapp messages (including images, videos, documents, and audio messages), search your contacts and send messages to either individuals or groups. You can also send media files including images, videos, documents, and audio messages.

It connects to your **personal WhatsApp account** directly via the Whatsapp web multidevice API (using the [whatsmeow](https://github.com/tulir/whatsmeow) library). All your messages are stored locally in a SQLite database and only sent to an LLM (such as Claude) when the agent accesses them through tools (which you control).

Here's an example of what you can do when it's connected to Claude.

![WhatsApp MCP](./example-use.png)

> _Caution:_ as with many MCP servers, the WhatsApp MCP is subject to [the lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/). This means that project injection could lead to private data exfiltration.

## Quick Start

The easiest way to get started is using Docker:

```bash
# Clone the repository
git clone https://github.com/voska/whatsapp-mcp.git
cd whatsapp-mcp

# Build and start the WhatsApp bridge
docker-compose up

# When you see the QR code, scan it with WhatsApp on your phone
# Then configure Claude Desktop (see below)
```

## Installation

### Prerequisites

For Docker setup (recommended):

- Docker and Docker Compose installed
- Anthropic Claude Desktop app (or Cursor)
- FFmpeg (_optional_) - Only needed for audio messages. If you want to send audio files as playable WhatsApp voice messages, they must be in `.ogg` Opus format. With FFmpeg installed, the MCP server will automatically convert non-Opus audio files. Without FFmpeg, you can still send raw audio files using the `send_file` tool.

For manual setup:

- Go 1.19+
- Python 3.6+ with UV package manager
- FFmpeg (_optional_) - Same as above

### Setup

#### Option 1: Docker (Recommended)

1. **Clone the repository**

   ```bash
   git clone https://github.com/voska/whatsapp-mcp.git
   cd whatsapp-mcp
   ```

2. **Build the Docker images**

   ```bash
   docker-compose build
   ```

3. **Start the WhatsApp bridge**

   ```bash
   docker-compose up
   ```

4. **Authenticate with WhatsApp**

   - When you first run it, you'll see a QR code in the terminal
   - Scan this QR code with WhatsApp on your phone (see [QR Code Authentication](#qr-code-authentication))

5. **Configure Claude Desktop** - Follow the [MCP Configuration](#mcp-configuration) section below

#### Option 2: Manual Setup

1. **Clone the repository**

   ```bash
   git clone https://github.com/voska/whatsapp-mcp.git
   cd whatsapp-mcp
   ```

2. **Install UV** (Python package manager)

   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```

3. **Run the WhatsApp bridge**

   ```bash
   cd whatsapp-bridge
   go run main.go
   ```

   Scan the QR code when it appears.

4. **Configure Claude Desktop** with manual setup instructions

### QR Code Authentication

When you first run the WhatsApp bridge, you'll need to link it to your WhatsApp account:

1. **Open WhatsApp on your phone**
2. **Go to Settings** (tap ⋮ on Android or Settings tab on iPhone)
3. **Select "Linked Devices"**
4. **Tap "Link a Device"**
5. **Scan the QR code** displayed in your terminal

The QR code looks like this in the terminal:

```
████████████████████████████████████████████
████ ▄▄▄▄▄ █▀█ █▄█▀█▀▄█ ▀▄▀▄▀ █ ▄▄▄▄▄ ████
████ █   █ █▀▀▀█ ▀▄█▀▀▀▀█▀ ▄ ▀█ █   █ ████
████ █▄▄▄█ █▀ █▀▀██▀▄▀ ▀█ ██▀▄█ █▄▄▄█ ████
... (more lines)
```

**Important Notes:**

- The QR code only appears on first run or after authentication expires (~20 days)
- If already authenticated, the bridge will connect automatically
- Keep the bridge running to maintain the WhatsApp connection

### MCP Configuration

Configure Claude Desktop or Cursor to use the MCP server:

#### For Docker Users

Copy this configuration and save it:

```json
{
  "mcpServers": {
    "whatsapp": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "-v",
        "{{PATH_TO_PROJECT}}/whatsapp-bridge/messages.db:/app/messages.db:ro",
        "walla-walla-mcp-server"
      ]
    }
  }
}
```

Replace `{{PATH_TO_PROJECT}}` with the full path to your whatsapp-mcp directory (run `pwd` in the project directory).

#### For Manual Setup Users

```json
{
  "mcpServers": {
    "whatsapp": {
      "command": "{{PATH_TO_UV}}",
      "args": [
        "--directory",
        "{{PATH_TO_PROJECT}}/whatsapp-mcp-server",
        "run",
        "main.py"
      ]
    }
  }
}
```

Replace:

- `{{PATH_TO_UV}}` with the output of `which uv`
- `{{PATH_TO_PROJECT}}` with the full path to your whatsapp-mcp directory

#### Save Configuration

**For Claude Desktop:**

- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

**For Cursor:**

- `~/.cursor/mcp.json`

After saving, restart Claude Desktop or Cursor.

## Docker Commands

### Basic Operations

```bash
# Start the bridge
docker-compose up -d

# View logs (and QR code)
docker-compose logs -f

# Stop the bridge
docker-compose down

# Rebuild after updates
docker-compose build
docker-compose up -d

# Reset everything (deletes all data)
docker-compose down -v
```

### Data Persistence

- WhatsApp session: Stored in `whatsapp-data` Docker volume
- Message history: Stored in `whatsapp-bridge/messages.db`

To backup your data:

```bash
# Backup messages
cp whatsapp-bridge/messages.db messages-backup.db

# Backup WhatsApp session
docker run --rm -v walla-walla_whatsapp-data:/data -v $(pwd):/backup alpine tar czf /backup/whatsapp-backup.tar.gz -C /data .
```

## Verifying Your Setup

1. **Check bridge status**

   ```bash
   docker-compose ps  # Should show "Up" status
   ```

2. **In Claude Desktop**
   - Look for the 🔌 icon and check if WhatsApp appears
   - Try: "Can you list my recent WhatsApp chats?"

## Usage

Once connected, Claude can help you:

- Search and read WhatsApp messages
- Send messages to contacts and groups
- Share files, images, videos, and voice messages
- Download media from conversations
- Search contacts by name or phone number

### Available Tools

Claude can access the following tools to interact with WhatsApp:

- **search_contacts**: Search for contacts by name or phone number
- **list_messages**: Retrieve messages with optional filters and context
- **list_chats**: List available chats with metadata
- **get_chat**: Get information about a specific chat
- **get_direct_chat_by_contact**: Find a direct chat with a specific contact
- **get_contact_chats**: List all chats involving a specific contact
- **get_last_interaction**: Get the most recent message with a contact
- **get_message_context**: Retrieve context around a specific message
- **send_message**: Send a WhatsApp message to a specified phone number or group JID
- **send_file**: Send a file (image, video, raw audio, document) to a specified recipient
- **send_audio_message**: Send an audio file as a WhatsApp voice message (requires the file to be an .ogg opus file or ffmpeg must be installed)
- **download_media**: Download media from a WhatsApp message and get the local file path

### Media Handling Features

The MCP server supports both sending and receiving various media types:

#### Media Sending

You can send various media types to your WhatsApp contacts:

- **Images, Videos, Documents**: Use the `send_file` tool to share any supported media type.
- **Voice Messages**: Use the `send_audio_message` tool to send audio files as playable WhatsApp voice messages.
  - For optimal compatibility, audio files should be in `.ogg` Opus format.
  - With FFmpeg installed, the system will automatically convert other audio formats (MP3, WAV, etc.) to the required format.
  - Without FFmpeg, you can still send raw audio files using the `send_file` tool, but they won't appear as playable voice messages.

#### Media Downloading

By default, just the metadata of the media is stored in the local database. The message will indicate that media was sent. To access this media you need to use the download_media tool which takes the `message_id` and `chat_jid` (which are shown when printing messages containing the media), this downloads the media and then returns the file path which can be then opened or passed to another tool.

## Architecture Overview

This application consists of two main components:

1. **Go WhatsApp Bridge** (`whatsapp-bridge/`): A Go application that connects to WhatsApp's web API, handles authentication via QR code, and stores message history in SQLite. It serves as the bridge between WhatsApp and the MCP server.

2. **Python MCP Server** (`whatsapp-mcp-server/`): A Python server implementing the Model Context Protocol (MCP), which provides standardized tools for Claude to interact with WhatsApp data and send/receive messages.

### Data Storage

- All message history is stored in a SQLite database within the `whatsapp-bridge/store/` directory
- The database maintains tables for chats and messages
- Messages are indexed for efficient searching and retrieval

### Technical Details

1. Claude sends requests to the Python MCP server
2. The MCP server queries the Go bridge for WhatsApp data or directly to the SQLite database
3. The Go accesses the WhatsApp API and keeps the SQLite database up to date
4. Data flows back through the chain to Claude
5. When sending messages, the request flows from Claude through the MCP server to the Go bridge and to WhatsApp

## Troubleshooting

### QR Code Issues

**Can't see the QR code:**

- Use `docker-compose logs -f` to view it
- Make sure your terminal window is wide enough
- Try `docker-compose restart` if it doesn't appear

**Authentication expired:**

- Simply restart the bridge to get a new QR code
- Scan it again with WhatsApp

### Connection Problems

**Bridge won't start:**

```bash
# Check if port 8080 is already in use
lsof -i :8080

# View detailed logs
docker-compose logs --tail=50
```

**MCP can't connect:**

- Ensure bridge is running: `docker-compose ps`
- Check your claude_desktop_config.json path is correct
- Try restarting Claude Desktop

### Data Issues

**Messages out of sync:**

```bash
# Reset and re-authenticate
docker-compose down -v
rm whatsapp-bridge/messages.db
docker-compose up
```

### Windows-Specific Issues

For manual setup on Windows, `go-sqlite3` requires CGO:

1. Install [MSYS2](https://www.msys2.org/) and add to PATH
2. Enable CGO:
   ```bash
   go env -w CGO_ENABLED=1
   go run main.go
   ```

## Security Notes

- All data is stored locally
- Messages are only sent to Claude when you explicitly use WhatsApp tools
- The bridge requires continuous connection to maintain authentication
- Consider the [security implications](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) of MCP servers

## Support

- Check logs: `docker-compose logs`
- File issues: [GitHub Issues](https://github.com/voska/whatsapp-mcp/issues)
- MCP docs: [Model Context Protocol](https://modelcontextprotocol.io)

## Credits

This project is a fork of the original [whatsapp-mcp](https://github.com/lharries/whatsapp-mcp) by [lharries](https://github.com/lharries). 

Major contributions in this fork:
- Fixed WebSocket connection issues by updating whatsmeow dependency
- Added Docker support for easy deployment
- Simplified setup process with Docker Compose
