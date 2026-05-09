# 🎬 arrstack-mcp

An [MCP](https://modelcontextprotocol.io/) server that gives AI assistants control over your **Sonarr**, **Radarr**, **Lidarr**, **Prowlarr**, **qBittorrent**, **SABnzbd**, and **Jellyfin** homelab media stack.

Works with **Claude Desktop**, **Cursor**, **VS Code Copilot**, **OpenClaw**, and any other MCP-compatible client.

## Demo

![Adding a movie with natural language](screenshots/DemoAddMovieScreenshot.png)

## Features

| Service | Tools |
|---------|-------|
| **Sonarr** | List series, search & add shows, upcoming episodes, download queue |
| **Radarr** | List movies, search & add movies, download queue |
| **Lidarr** | List artists, search & add artists/albums, queue, missing search |
| **Prowlarr** | List/test indexers, search releases, health check |
| **qBittorrent** | List/pause/resume/delete torrents, add magnets, transfer stats |
| **SABnzbd** | Queue, history, status, pause/resume, add NZB url, speed limit |
| **Jellyfin** | List libraries, recent additions, system info |

Only configure the services you use — unconfigured services are gracefully skipped.

## Quick Start

### Option 1: Claude Desktop / Cursor / VS Code (stdio)

1. Install dependencies:

   ```bash
   pip install "mcp[cli]>=1.9.0" httpx
   ```

2. Add to your MCP client config (e.g. `claude_desktop_config.json`):

   ```json
   {
     "mcpServers": {
       "arrstack": {
         "command": "python",
         "args": ["/path/to/arrstack-mcp/server.py"],
         "env": {
           "SONARR_URL": "http://localhost:8989",
           "SONARR_API_KEY": "your-api-key",
           "RADARR_URL": "http://localhost:7878",
           "RADARR_API_KEY": "your-api-key",
           "QBT_URL": "http://localhost:8080",
           "QBT_USER": "admin",
           "QBT_PASS": "your-password",
           "JELLYFIN_URL": "http://localhost:8096"
         }
       }
     }
   }
   ```

3. Restart your MCP client. Done!

### Option 2: Docker (HTTP transport)

For remote setups or when running alongside your *arr stack:

```bash
git clone https://github.com/ct4nk3r/arrstack-mcp.git
cd arrstack-mcp
cp .env.example .env
# Edit .env with your service URLs and API keys
docker compose up -d
```

The server runs on port `8000` with Streamable HTTP transport.

#### Connect to OpenClaw

```bash
openclaw mcp set arrstack '{"url":"http://arrstack-mcp:8000/mcp","transport":"streamable-http"}'
```

#### Connect to other HTTP MCP clients

Point your client to `http://<host>:8000/mcp` using Streamable HTTP transport.

### Option 3: Docker on the same network as your *arr stack

If your media services run in Docker, add `arrstack-mcp` to the same network:

```yaml
services:
  arrstack-mcp:
    build: .
    container_name: arrstack-mcp
    ports:
      - "8000:8000"
    environment:
      - SONARR_URL=http://sonarr:8989
      - SONARR_API_KEY=your-key
      - RADARR_URL=http://radarr:7878
      - RADARR_API_KEY=your-key
      - QBT_URL=http://qbittorrent:8080
      - QBT_USER=admin
      - QBT_PASS=your-password
      - JELLYFIN_URL=http://jellyfin:8096
    networks:
      - your-media-network
```

## Configuration

All configuration is done via environment variables:

| Variable | Required | Description |
|----------|----------|-------------|
| `SONARR_URL` | No | Sonarr base URL (e.g. `http://localhost:8989`) |
| `SONARR_API_KEY` | If Sonarr | Sonarr API key (Settings → General) |
| `RADARR_URL` | No | Radarr base URL (e.g. `http://localhost:7878`) |
| `RADARR_API_KEY` | If Radarr | Radarr API key (Settings → General) |
| `LIDARR_URL` | No | Lidarr base URL (e.g. `http://localhost:8686`) |
| `LIDARR_API_KEY` | If Lidarr | Lidarr API key (Settings → General) |
| `QBT_URL` | No | qBittorrent Web UI URL (e.g. `http://localhost:8080`) |
| `QBT_USER` | If qBt | qBittorrent username (default: `admin`) |
| `QBT_PASS` | If qBt | qBittorrent password |
| `JELLYFIN_URL` | No | Jellyfin base URL (e.g. `http://localhost:8096`) |
| `JELLYFIN_API_KEY` | No | Jellyfin API key (optional, for authenticated endpoints) |
| `PROWLARR_URL` | No | Prowlarr base URL (e.g. `http://localhost:9696`) |
| `PROWLARR_API_KEY` | If Prowlarr | Prowlarr API key (Settings → General) |
| `SAB_URL` | No | SABnzbd base URL (e.g. `http://localhost:8080`) |
| `SAB_API_KEY` | If SABnzbd | SABnzbd API key (Config → General → API Key) |
| `LOG_LEVEL` | No | Python logging level (default: `INFO`; e.g. `DEBUG`, `WARNING`) |

## Available Tools

### Sonarr (TV Shows)

| Tool | Description |
|------|-------------|
| `sonarr_list_series` | List all series with episode counts and disk usage |
| `sonarr_get_series` | Get detailed info about a specific series |
| `sonarr_search` | Search for new shows to add |
| `sonarr_add_series` | Add a show by TVDB ID |
| `sonarr_upcoming` | Show upcoming episodes |
| `sonarr_queue` | Show current download queue |

### Radarr (Movies)

| Tool | Description |
|------|-------------|
| `radarr_list_movies` | List all movies with download status |
| `radarr_get_movie` | Get detailed info about a specific movie |
| `radarr_search` | Search for new movies to add |
| `radarr_add_movie` | Add a movie by TMDB ID |
| `radarr_queue` | Show current download queue |

### Lidarr (Music)

| Tool | Description |
|------|-------------|
| `lidarr_list_artists` | List all artists with album/track counts and disk usage |
| `lidarr_get_artist` | Get detailed info about a specific artist |
| `lidarr_search` | Search for artists to add |
| `lidarr_search_album` | Search for albums in metadata |
| `lidarr_add_artist` | Add an artist by name (requires quality + metadata profile + root folder) |
| `lidarr_list_quality_profiles` | List quality profiles |
| `lidarr_list_metadata_profiles` | List metadata profiles |
| `lidarr_list_root_folders` | List root folders with free space |
| `lidarr_queue` | Show current download queue |
| `lidarr_delete_queue_item` | Remove an item from the queue (optionally blocklist) |
| `lidarr_search_missing` | Trigger search for all missing albums |

### Prowlarr (Indexers)

| Tool | Description |
|------|-------------|
| `prowlarr_list_indexers` | List all indexers with status |
| `prowlarr_test_indexer` | Test a specific indexer connection |
| `prowlarr_test_all_indexers` | Test all enabled indexers |
| `prowlarr_search` | Search across indexers for releases |
| `prowlarr_health` | Check system health warnings |

### qBittorrent (Downloads)

| Tool | Description |
|------|-------------|
| `qbt_list_torrents` | List torrents with progress and speed |
| `qbt_torrent_details` | Get detailed torrent info |
| `qbt_add_magnet` | Add a magnet link |
| `qbt_pause` | Pause a torrent |
| `qbt_resume` | Resume a torrent |
| `qbt_delete` | Delete a torrent (optionally with files) |
| `qbt_transfer_info` | Global transfer statistics |

### SABnzbd (Usenet Downloads)

| Tool | Description |
|------|-------------|
| `sab_queue` | Show the current download queue |
| `sab_history` | Show download history |
| `sab_status` | Show full server status (disk, speed, etc.) |
| `sab_pause` | Pause the entire queue |
| `sab_resume` | Resume the entire queue |
| `sab_pause_job` | Pause a specific queue item by NZO id |
| `sab_resume_job` | Resume a specific queue item by NZO id |
| `sab_delete_job` | Delete a queue item (optionally with files) |
| `sab_add_url` | Add an NZB by URL (with optional category/priority) |
| `sab_speed_limit` | Set the global speed limit (0..100% of configured max) |

### Jellyfin (Media Server)

| Tool | Description |
|------|-------------|
| `jellyfin_libraries` | List media libraries |
| `jellyfin_recent` | Recently added items |
| `jellyfin_system_info` | Server version and system info |

## Transport Options

```bash
# stdio (default) — for Claude Desktop, Cursor, VS Code
python server.py

# Streamable HTTP — for Docker / remote
python server.py --transport streamable-http --port 8000

# SSE — legacy HTTP transport
python server.py --transport sse --port 8000
```

## Finding Your API Keys

- **Sonarr**: Settings → General → API Key
- **Radarr**: Settings → General → API Key
- **Lidarr**: Settings → General → API Key
- **Prowlarr**: Settings → General → API Key
- **qBittorrent**: Settings → Web UI → Authentication
- **SABnzbd**: Config → General → API Key
- **Jellyfin**: Dashboard → API Keys → Add

## Security

The HTTP/SSE transports listen on `0.0.0.0:8000` by default to match the
upstream community behavior, but the MCP protocol itself has **no built-in
authentication**. Anyone who can reach the port can call your *arr stack with
your API keys. Choose one of the following deployment patterns:

- **Reverse proxy with auth (recommended for LAN/WAN exposure).** Bind the
  server to `127.0.0.1` (`--host 127.0.0.1`) and front it with Caddy / nginx /
  Traefik enforcing basic auth, OAuth2-proxy, or mTLS. Configure
  `TransportSecuritySettings(allowed_hosts=[...])` for your public hostname if
  you need to relax the default DNS-rebinding protection.
- **Tailscale-only exposure.** Bind to the Tailscale interface (or
  `127.0.0.1` and run `tailscale serve`) so only devices on your tailnet can
  reach the MCP server.
- **stdio for desktop clients.** When using Claude Desktop / Cursor / VS Code
  on the same machine, prefer the default `stdio` transport — no network
  surface at all.

Other security notes:

- DNS-rebinding protection is **enabled** by default for HTTP/SSE transports.
  If you serve under a custom hostname, set `allowed_hosts` accordingly in
  `TransportSecuritySettings` (see `server.py`).
- The Docker image runs as a non-root `appuser` (uid 1000).
- API keys are read from environment variables and are never logged. Set
  `LOG_LEVEL=DEBUG` for verbose request logging (paths and methods only — no
  headers or bodies). Default level is `INFO`.

## License

MIT
