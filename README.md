# MCP Weather Server

A simple MCP server that provides hourly weather forecasts using the AccuWeather API.

## Setup

1. Install dependencies using `uv`:
```bash
uv venv
uv sync
```

2. Create a `.env` file with your AccuWeather API key:
```
ACCUWEATHER_API_KEY=your_api_key_here
```

You can get an API key by registering at [AccuWeather API](https://developer.accuweather.com/).

## Running the Server

### STDIO Mode (Default)

For local MCP client connections via stdio:

```json
{
    "mcpServers": {
        "weather": {
            "command": "uvx",
            "args": ["--from", "git+https://github.com/adhikasp/mcp-weather.git", "mcp-weather"],
            "env": {
                "ACCUWEATHER_API_KEY": "your_api_key_here"
            }
        }
    }
}
```

### HTTP Streaming Mode

To run the server as a streamable HTTP server:

**Option 1: Using the command line**
```bash
python -m mcp_weather.weather --http
```

**Option 2: Using the entry point**
```bash
mcp-weather-http
```

**Option 3: Using environment variables**
```bash
export MCP_HOST=0.0.0.0
export MCP_PORT=8000
python -m mcp_weather.weather --http
```

The server will start on `http://0.0.0.0:8000` (or your specified host/port) and expose the MCP endpoint at `/mcp`. The streamable-http transport supports:
- **JSON responses** for standard requests
- **Server-Sent Events (SSE)** for streaming long-running operations
- **Remote access** via HTTP endpoints
- **Horizontal scaling** with load balancing

**Connecting to HTTP Server:**

For MCP clients that support HTTP transport, configure the connection URL:
```
http://localhost:8000/mcp
```

**Agent UI Configuration:**

When configuring your MCP server in the Agent UI (e.g., Cursor, Claude Desktop, etc.):

- **URL**: Provide the full endpoint URL (e.g., `http://localhost:8000/mcp` or `https://your-ngrok-url.ngrok.io/mcp`)
- **Authentication**: **Not required by default** - No API keys, tokens, or scopes need to be configured in the Agent UI
- **Headers**: No custom headers needed for basic operation

The MCP protocol handles communication automatically. Your AccuWeather API key is stored server-side and never needs to be shared with the client.

**Note**: If you implement custom authentication (API keys, tokens) at the HTTP layer for production use, you would configure those in your server code, not in the Agent UI. The Agent UI would then need to include those credentials in HTTP requests, but this is not needed for basic operation.

**Production Deployment:**

For production, you can use environment variables or configure a reverse proxy:
```bash
# Set custom host and port
export MCP_HOST=0.0.0.0
export MCP_PORT=8080
mcp-weather-http
```

### Exposing via Public URL (Tunneling)

To make your local server accessible via a public URL, use a tunneling service:

#### Option 1: ngrok (Recommended)

1. **Install ngrok on Windows**:

   **Option A: Using winget (Windows 10/11 - Recommended)**
   ```powershell
   winget install ngrok.ngrok
   ```

   **Option B: Using Chocolatey** (if you have Chocolatey installed)
   ```powershell
   choco install ngrok
   ```

   **Option C: Manual Download**
   - Go to [ngrok.com/download](https://ngrok.com/download)
   - Download the Windows ZIP file
   - Extract `ngrok.exe` to a folder (e.g., `C:\ngrok`)
   - Add that folder to your PATH, or run ngrok from that folder
   - Or simply run: `.\ngrok.exe http 8000` from the extracted folder

   **Option D: Using Scoop** (if you have Scoop installed)
   ```powershell
   scoop install ngrok
   ```

   **After installation, verify it works:**
   ```powershell
   ngrok version
   ```

2. **Start your MCP server**:
   ```bash
   python -m mcp_weather.weather --http
   ```

3. **Create a tunnel** (in a separate terminal):
   ```bash
   ngrok http 8000
   ```

4. **Use the public URL**: ngrok will provide a public URL like `https://abc123.ngrok-free.dev`. 

   **Important**: The MCP endpoint is at `/mcp`, not the root URL. Use:
   ```
   https://abc123.ngrok-free.dev/mcp
   ```
   
   **Note about ngrok warning page**: Free ngrok accounts show a warning page when visiting the root URL. This is normal and expected. The `/mcp` endpoint will work correctly for MCP clients. To remove the warning page, you can:
   - Upgrade to a paid ngrok account
   - Set the `ngrok-skip-browser-warning` header in your MCP client (if supported)
   - Use a custom domain with a paid account

   **For a custom domain** (requires paid ngrok account):
   ```bash
   ngrok http 8000 --domain=your-custom-domain.ngrok.io
   ```

#### Option 2: Cloudflare Tunnel (cloudflared)

1. **Install cloudflared**: Download from [developers.cloudflare.com](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/)

2. **Start your MCP server**:
   ```bash
   python -m mcp_weather.weather --http
   ```

3. **Create a tunnel** (in a separate terminal):
   ```bash
   cloudflared tunnel --url http://localhost:8000
   ```

4. **Use the public URL**: Cloudflare will provide a public URL like `https://random-subdomain.trycloudflare.com`. Your MCP endpoint will be:
   ```
   https://random-subdomain.trycloudflare.com/mcp
   ```

#### Option 3: localtunnel

1. **Install localtunnel**:
   ```bash
   npm install -g localtunnel
   ```

2. **Start your MCP server**:
   ```bash
   python -m mcp_weather.weather --http
   ```

3. **Create a tunnel** (in a separate terminal):
   ```bash
   lt --port 8000
   ```

4. **Use the public URL**: localtunnel will provide a public URL like `https://random-name.loca.lt`. Your MCP endpoint will be:
   ```
   https://random-name.loca.lt/mcp
   ```

**Security Note**: When exposing your server publicly, consider:
- Adding authentication/API keys if your MCP server handles sensitive data
- Using HTTPS (all tunneling services above provide HTTPS)
- Limiting access to specific IPs if possible
- Monitoring usage and rate limiting

## API Usage

### Get Hourly Weather Forecast

Response:
```json
{
    "location": "Jakarta",
    "location_key": "208971",
    "country": "Indonesia",
    "current_conditions": {
        "temperature": {
            "value": 32.2,
            "unit": "C"
        },
        "weather_text": "Partly sunny",
        "relative_humidity": 75,
        "precipitation": false,
        "observation_time": "2024-01-01T12:00:00+07:00"
    },
    "hourly_forecast": [
        {
            "relative_time": "+1 hour",
            "temperature": {
                "value": 32.2,
                "unit": "C"
            },
            "weather_text": "Partly sunny",
            "precipitation_probability": 40,
            "precipitation_type": "Rain",
            "precipitation_intensity": "Light"
        }
    ]
}
```

The API provides:
- Current weather conditions including temperature, weather description, humidity, and precipitation status
- 12-hour forecast with hourly data including:
  - Relative time from current time
  - Temperature in Celsius
  - Weather description
  - Precipitation probability, type, and intensity