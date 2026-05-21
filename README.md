<div align="center" style="display: flex; align-items: center; justify-content: center; gap: 10px;">
<img src="https://upload.wikimedia.org/wikipedia/commons/8/84/Spotify_icon.svg" width="30" height="30">
<h1>Spotify MCP Server</h1>
</div>

A lightweight [Model Context Protocol (MCP)](https://modelcontextprotocol.io) server that lets AI assistants like Claude, Cursor, and Cline control Spotify playback and manage playlists.

> Fork of [marcelmarais/spotify-mcp-server](https://github.com/marcelmarais/spotify-mcp-server) with additional tools, Spotify February 2026 API migration fixes, and playlist cover image editing.

## Quick Start

```bash
# 1. Clone and build
git clone https://github.com/oscarmorrison/spotify-mcp-server.git
cd spotify-mcp-server
npm install && npm run build

# 2. Add Spotify credentials
cp spotify-config.example.json spotify-config.json
# edit spotify-config.json with your clientId / clientSecret

# 3. Authenticate (opens browser, saves tokens back to spotify-config.json)
npm run auth
```

Then add the server to your MCP client (see [Client setup](#client-setup) below):

```json
{
  "mcpServers": {
    "spotify": {
      "command": "node",
      "args": ["/absolute/path/to/spotify-mcp-server/build/index.js"]
    }
  }
}
```

**Requirements:** Node.js v16+, a Spotify Premium account, and a [Spotify developer app](#creating-a-spotify-developer-application) with redirect URI `http://127.0.0.1:8888/callback`.

## Example prompts

- _"Play Elvis's first song"_
- _"Create a Taylor Swift / Slipknot fusion playlist"_
- _"Copy all the techno tracks from my workout playlist to my work playlist"_
- _"Turn the volume down a bit"_

## Tools

<details>
<summary><strong>Playback & queue</strong> — play, pause, skip, seek, shuffle, repeat, volume, devices, queue</summary>

| Tool | Description |
| --- | --- |
| `playMusic` | Play a track/album/artist/playlist by URI or `type` + `id`. Optional `deviceId`. |
| `pausePlayback` / `resumePlayback` | Pause or resume current playback. |
| `skipToNext` / `skipToPrevious` | Move within the current queue. |
| `seekToPosition` | Seek to a timestamp in the current track. |
| `setRepeatMode` | Set repeat to `track`, `context`, or `off`. |
| `toggleShuffle` | Toggle shuffle on/off. |
| `setVolume` | Set absolute volume `0–100` (Premium). |
| `adjustVolume` | Adjust volume by relative `-100..100` (Premium). |
| `addToQueue` | Queue a track/album/artist/playlist by URI or `type` + `id`. |
| `getQueue` | Current track + upcoming items (limit 1–50). |
| `getNowPlaying` | Current track, device, volume, shuffle/repeat state. |
| `getAvailableDevices` | List Spotify Connect devices. |
| `transferPlayback` | Move playback to another device. |

</details>

<details>
<summary><strong>Library & search</strong> — search, liked songs, saved albums/shows/episodes/audiobooks, recently played, top items, followed artists</summary>

| Tool | Description |
| --- | --- |
| `searchSpotify` | Search tracks/albums/artists/playlists (max 10 results — Feb 2026 API cap). |
| `getRecentlyPlayed` | Recently played tracks. |
| `getUsersSavedTracks` | Liked Songs (paginated, 1–50). |
| `removeUsersSavedTracks` | Remove up to 40 tracks from Liked Songs. |
| `getCurrentUserProfile` | The authenticated user's profile (`GET /me`). |
| `getUserTopItems` | Top tracks or artists. |
| `getFollowedArtists` | Artists the user follows. |
| `followArtists` | Follow or unfollow artists. |
| `getSavedShows` / `getSavedEpisodes` / `getSavedAudiobooks` | Library content for podcasts and audiobooks. |
| Metadata lookups | `getArtists`, `getTracks`, `getShows`, `getEpisodes`, `getAudiobooks`, `getChapters`. |

</details>

<details>
<summary><strong>Albums</strong> — details, tracks, save/remove, saved status</summary>

| Tool | Description |
| --- | --- |
| `getAlbums` | Details for one ID or up to 20 IDs. |
| `getAlbumTracks` | Paginated tracks (1–50) for an album. |
| `saveOrRemoveAlbumForUser` | Save or remove up to 20 albums in "Your Music". |
| `checkUsersSavedAlbums` | Check saved status for up to 20 album IDs. |

</details>

<details>
<summary><strong>Playlists</strong> — list, create, edit metadata, add/remove/reorder tracks, custom cover image</summary>

| Tool | Description |
| --- | --- |
| `getMyPlaylists` | The user's playlists (paginated). |
| `getPlaylist` | Playlist details: name, owner, track count, description, URL. |
| `getPlaylistTracks` | Paginated tracks for a playlist. |
| `createPlaylist` | Create a new playlist (`name`, optional `description`, `public`). |
| `updatePlaylist` | Update `name` / `description` / `public` / `collaborative`. |
| `addTracksToPlaylist` | Add track URIs (optional `position`). |
| `removeTracksFromPlaylist` | Remove up to 100 tracks. |
| `reorderPlaylistItems` | Move a range to a new position. |
| `updatePlaylistImage` | Upload a custom JPEG cover (base64, <256 KB; requires `ugc-image-upload` scope). |

</details>

## Setup

### Creating a Spotify Developer Application

1. Open the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard/) and log in.
2. Click **Create an App** and fill in the name and description.
3. From the app's page, copy the **Client ID** and reveal the **Client Secret**.
4. Click **Edit Settings** and add `http://127.0.0.1:8888/callback` as a Redirect URI.

### Configure credentials

```bash
cp spotify-config.example.json spotify-config.json
```

```json
{
  "clientId": "your-client-id",
  "clientSecret": "your-client-secret",
  "redirectUri": "http://127.0.0.1:8888/callback"
}
```

### Authenticate

```bash
npm run auth
```

The script prints an authorization URL — open it, approve, and Spotify redirects back to the local callback. Access and refresh tokens are written back to `spotify-config.json`. **Tokens refresh automatically;** re-run `npm run auth` only if the refresh fails.

### Client setup

**Claude Desktop** — add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "spotify": {
      "command": "node",
      "args": ["/absolute/path/to/spotify-mcp-server/build/index.js"]
    }
  }
}
```

**Cursor** — `Cursor Settings` → **MCP** (⌘⇧J), add a server with command:

```bash
node /absolute/path/to/spotify-mcp-server/build/index.js
```

**Cline (VS Code)** — in `cline_mcp_settings.json`:

```json
{
  "mcpServers": {
    "spotify": {
      "command": "node",
      "args": ["/absolute/path/to/spotify-mcp-server/build/index.js"],
      "autoApprove": ["getNowPlaying", "getRecentlyPlayed"]
    }
  }
}
```

Add more tools to `autoApprove` to skip per-call confirmation.

## Changes from upstream

**Spotify February 2026 API migration**
- `getPlaylistTracks` updated to `GET /playlists/{id}/items`
- Playlist and library `PUT`/`DELETE` endpoints now use query params instead of JSON bodies
- Search results capped at 10 per updated API limits
- Restored automatic token refresh

**New tools** — `getCurrentUserProfile`, `getUserTopItems`, `getFollowedArtists`, `followArtists`, `getSavedAudiobooks`, `getSavedEpisodes`, `getSavedShows`, metadata lookups for artists/tracks/shows/episodes/audiobooks/chapters, `seekToPosition`, `setRepeatMode`, `toggleShuffle`, `transferPlayback`, `updatePlaylistImage`.

**OAuth scopes added** — `user-follow-read`, `user-follow-modify`, `user-top-read`, `ugc-image-upload`.
