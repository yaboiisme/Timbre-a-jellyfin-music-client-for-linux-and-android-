

<h1 align="center">Timbre</h1>

<p align="center">
  A warm, fast music player for the Jellyfin library you already own.<br />
  Built with Rust, Tauri and React.
</p>

<p align="center">
  <img src="docs/screenshots/02-home.png" alt="Timbre home screen" width="920" />
</p>

Timbre is a focused alternative to the usual music app: no feeds, no recommendations, no
algorithm. Sign in to your own Jellyfin server and browse your albums, artists, playlists and
favourites in a calm, dark interface built for long listening sessions.

It runs as a native desktop app through Tauri, where Rust handles authentication and API
requests, and it also builds as a static web app for GitHub Pages.

## Features

- **Your library, front and centre** — home with continue listening, recent additions and
  on-rotation albums; full album, artist, song, playlist and favourites views.
- **Search across everything** — albums, artists, tracks and playlists from one field.
- **A real queue** — play an album or playlist, drag to reorder, remove tracks, shuffle,
  repeat one/all, seek and control volume.
- **Playback that remembers** — resume positions come from Jellyfin, progress is reported back
  to the server, and media keys work through the Media Session API.
- **Favourites** — heart tracks from the player, album pages or search results.
- **Codec fallback** — if direct play fails, Timbre retries through Jellyfin's transcoder.
- **Desktop and web** — the same React app runs inside a Rust/Tauri shell or as a static site.
- **Current Jellyfin auth** — API calls use the modern `Authorization: MediaBrowser` scheme, so
  Timbre works with Jellyfin 12 and older 10.x servers (including servers with legacy
  authorization disabled).
- **Responsive** — a collapsible sidebar on desktop and a two-row player on phones.
- **No telemetry** — requests go only to the Jellyfin server you choose.
- 
## Using your own Jellyfin server

1. Enter the server's base URL, for example `https://jellyfin.example.com`. A bare hostname such
   as `jellyfin.example.com` is normalised to `http://`; the `/web` path is stripped.
2. Enter your Jellyfin username and password.
3. Timbre exchanges the password for a Jellyfin access token. The password itself is never
   stored.

The native desktop build is the best option when your server is not reachable from a browser
origin, or when you do not control its CORS headers.

## Desktop app

### Prerequisites

- **Rust** 1.77.2 or newer (`rustup` recommended)
- **Linux:** WebKitGTK 4.1 and the usual Tauri build dependencies, for example on
  Ubuntu/Debian:

  ```bash
  sudo apt install libwebkit2gtk-4.1-dev build-essential curl wget file \
    libxdo-dev libssl-dev libayatana-appindicator3-dev librsvg2-dev
  ```

## Security and privacy

- Timbre talks directly to the Jellyfin server you enter. There is no Timbre account, backend or
  analytics service.
- Your Jellyfin password is used once to obtain an access token and is not persisted.
- The native app stores the token in a user-private config file (`0600` on Unix). The web app
  stores it in `localStorage`; protect that browser profile and use HTTPS.
- Logging out removes the local token. To invalidate it on the server as well, revoke the device
  in Jellyfin's **Dashboard → Devices**.
- The desktop window currently ships without a restrictive CSP because it must be able to load
  media and images from any user-supplied server. Treat it as a normal desktop application and
  only connect it to servers you trust.

## License

[MIT](LICENSE) © 2026 Timbre contributors
