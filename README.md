<p align="center">
  <img src="assets/app-icon.svg" width="72" height="72" alt="Timbre app icon" />
</p>

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

### Screenshots

| Library | Album | Player |
| --- | --- | --- |
| ![Albums](docs/screenshots/03-albums.png) | ![Album detail](docs/screenshots/04-album.png) | ![Now playing](docs/screenshots/05-player.png) |

| Queue | Mobile | Sign in |
| --- | --- | --- |
| ![Queue](docs/screenshots/07-queue.png) | ![Mobile layout](docs/screenshots/06-mobile.png) | ![Login](docs/screenshots/01-login.png) |

## Quick start

You need **Node.js 20+** and npm. Rust is only required for the desktop build.

```bash
git clone https://github.com/YOUR_NAME/timbre.git
cd timbre
npm install
npm run dev
```

Open <http://localhost:1420>, enter your Jellyfin server address and sign in with your own
Jellyfin account.

### Try it without a Jellyfin server

A small mock Jellyfin server is included for development. It has a fictional library, generated
cover art and short generated audio files.

```bash
npm run mock     # http://localhost:8097
npm run dev      # in another terminal
```

Sign in with:

- **Server:** `http://localhost:8097`
- **Username:** anything (for example `Yassin`)
- **Password:** anything (for example `timbre`)

The mock is a development fixture. It keeps data in memory, accepts any credentials and serves
24-second WAV files, so track durations in the UI describe the fictional library rather than the
generated audio.

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

- **macOS:** Xcode Command Line Tools
- **Windows:** Microsoft Edge WebView2 (installed by default on Windows 10/11)

### Run and build

```bash
npm run desktop:dev      # Tauri dev window, with Vite hot reload
npm run desktop:build    # native installers in src-tauri/target/release/bundle
```

In desktop mode, network calls to Jellyfin are made by Rust through `reqwest`. The webview never
needs CORS permission, and the access token is written to a private session file:

| Platform | Location |
| --- | --- |
| Linux | `~/.config/app.timbre.player/session.json` |
| macOS | `~/Library/Application Support/app.timbre.player/session.json` |
| Windows | `%APPDATA%\app.timbre.player\session.json` |

On Unix the file is created with `0600` permissions. Passwords are never written to disk.

## Web app and GitHub Pages

```bash
npm run build     # static files in dist/
npm run preview   # serve the production build locally
```

The build uses a relative base and `HashRouter`, so the same `dist/` works at a domain root or
under a repository sub-path such as `https://name.github.io/timbre/`.

### CORS is required for the web build

A browser will only let the web app call your Jellyfin API if the server allows the app's origin.
Jellyfin does not expose a per-origin CORS setting in its dashboard, so the usual solution is a
small reverse proxy in front of Jellyfin. Example nginx configuration:

```nginx
server {
    listen 443 ssl http2;
    server_name jellyfin.example.com;

    # ... your existing TLS configuration ...

    location / {
        proxy_pass http://127.0.0.1:8096;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # GitHub Pages project sites share an origin: https://NAME.github.io
        add_header Access-Control-Allow-Origin "https://NAME.github.io" always;
        add_header Access-Control-Allow-Methods "GET, POST, DELETE, OPTIONS" always;
        add_header Access-Control-Allow-Headers "Authorization, Content-Type" always;
        add_header Access-Control-Expose-Headers "Content-Length, Content-Range" always;

        if ($request_method = OPTIONS) {
            return 204;
        }
    }
}
```

Notes:

- The CORS origin has **no path**. For `https://name.github.io/timbre/` use
  `https://name.github.io`.
- GitHub Pages is served over HTTPS. A browser will block requests to an `http://` Jellyfin
  server from an HTTPS page as mixed content, so use HTTPS for the server too.
- The web build stores its Jellyfin access token in `localStorage` under `timbre.session`.
  That is convenient but not as strong as the desktop session file; only use the web build on a
  device and origin you trust.
- API requests send the token in the `Authorization` header. Cover art and audio URLs use
  Jellyfin's `ApiKey` query parameter because `<img>` and `<audio>` elements cannot set headers.
  Query parameters can appear in reverse-proxy logs, so keep access logs private.

### Deploy the page

A workflow is included at `.github/workflows/deploy.yml`. To publish:

1. Push the repository to GitHub.
2. Open **Settings → Pages** and set **Build and deployment → Source** to **GitHub Actions**.
3. Push to `main` (or run the workflow manually). The site will be published at
   `https://NAME.github.io/REPO/`.

No change to `vite.config.ts` is needed for a project sub-path.

## Development

| Command | What it does |
| --- | --- |
| `npm run dev` | Vite dev server on port 1420 |
| `npm run build` | Type-check and build the web app into `dist/` |
| `npm run preview` | Preview the production build |
| `npm run typecheck` | TypeScript only |
| `npm run mock` | Run the mock Jellyfin server on port 8097 |
| `npm run desktop:dev` | Run the Tauri desktop app |
| `npm run desktop:build` | Build native bundles |

Rust tests:

```bash
cd src-tauri
cargo test
```

The CI workflow runs the web build and the Rust tests on every push and pull request.

### Project layout

```text
src/                  React application
  components/         Player, queue, cards, track lists, shell
  views/              Home, albums, artists, songs, playlists, favourites, search, login
  lib/api.ts          Native (Tauri invoke) and web (fetch) Jellyfin adapters
  lib/queries.ts      TanStack Query hooks for the Jellyfin API
  store/              Zustand auth, player and UI state
src-tauri/            Rust desktop shell
  src/jellyfin.rs     Jellyfin client, URL normalisation, request proxy
  src/session.rs      Private on-disk session storage
scripts/              Mock Jellyfin server used for development
docs/screenshots/     README screenshots
```

## Keyboard shortcuts

| Key | Action |
| --- | --- |
| `Space` | Play / pause |
| `Shift` + `→` | Next track |
| `Shift` + `←` | Previous track |
| `M` | Mute / unmute |
| `/` | Focus search |
| `Esc` | Close the queue |

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

## Troubleshooting

### `401 Unauthorized` after a successful sign-in

Jellyfin 12 removed the old `X-Emby-Token` header and lowercase `api_key` query parameter. Current
Timbre uses the modern `Authorization: MediaBrowser ... Token="..."` scheme instead, which works
on Jellyfin 10.8+ as well as 12.0+.

If you still see a 401:

- Make sure you are running a build that includes the modern auth code, then sign out and in again.
- For the web build, make sure the reverse proxy forwards the `Authorization` header and allows it
  in CORS (`Access-Control-Allow-Headers "Authorization, Content-Type"`).
- Do not put a legacy `X-Emby-Token` header in front of the request. Sending legacy and modern
  credentials together can also be rejected.
- If you disabled **Allow legacy authorization** on Jellyfin 10.11, Timbre's modern header still
  works; the old header would not.

## Limitations

- No offline downloads, lyrics, scrobbling or playlist editing yet.
- The web build depends on CORS and HTTPS from your Jellyfin deployment.
- Tested against the included mock server and Jellyfin's standard API; very old Jellyfin
  versions may not implement every endpoint Timbre uses.
- Self-signed HTTPS certificates are rejected by the desktop HTTP client unless trusted by the
  operating system.

## License

[MIT](LICENSE) © 2026 Timbre contributors
