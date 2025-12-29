# bettervlsub (modern VLSub)

VLC Lua extension to search and download subtitles from **OpenSubtitles.com** using the current REST API. This fork refreshes the original VLSub codebase to keep it responsive on modern VLC releases and to work with the new OpenSubtitles authentication model.

## Compatibility

* Designed for VLC 3.x and tested with recent VLC 4.0 nightlies.
* Uses the VLC Lua *extension* API (`View > VLsub`), no external Lua modules required.
* Network operations run with explicit timeouts and backoff to avoid the “Not Responding” freezes reported on older builds.

## Installation

Install `vlsub.lua` and the `locale` directory into your VLC extensions folder (create the `extensions` directory if it does not exist):

* **Windows (all users):** `%ProgramFiles%\VideoLAN\VLC\lua\extensions\`
* **Windows (current user):** `%APPDATA%\vlc\lua\extensions\`
* **Linux (all users):** `/usr/lib/vlc/lua/extensions/`
* **Linux (current user):** `~/.local/share/vlc/lua/extensions/`
* **macOS (all users):** `/Applications/VLC.app/Contents/MacOS/share/lua/extensions/`
* **macOS (current user):** `/Users/<you>/Library/Application Support/org.videolan.vlc/lua/extensions/`

Translations live under `locale/` and can be dropped alongside `vlsub.lua` in the working directory shown in the Config view (“Show config”).

## Configuration (new OpenSubtitles REST API)

Open the extension, switch to **Config**, and set:

* **OpenSubtitles API key (required):** create one from your [OpenSubtitles.com profile](https://www.opensubtitles.com/en/consumers). The extension will not search without it.
* **User access token (optional):** paste a personal token for higher download quotas. If omitted, anonymous limits apply.
* **Request timeout / retry attempts:** tune if you are on a slow or flaky network.
* Other legacy options (default language, working directory, add language code to filename, remove tags) remain available.

No credentials are hardcoded; everything is stored in the VLSub config file within the working directory.

## Usage

1. Start VLC and play your video.
2. Open **View > VLsub** (or **VLC > Extension > VLSub** on macOS).
3. Pick a subtitle language and search either by **hash** (best sync) or **name**.
4. Select a result and click **Download selection**.
   * If “Manual download” is selected or there is no active video input, a browser link is shown instead.
5. The subtitle is saved next to the video when possible; otherwise, VLSub falls back to the configured working directory.

## Troubleshooting

* **API errors (401/403):** confirm the API key and, if required, your user token in Config.
* **Rate limits (429):** VLSub now backs off automatically. Wait a few seconds and retry.
* **Timeouts or empty results:** check your network and increase the timeout slider in Config.
* **Non‑ASCII Windows paths:** VLC can still struggle to write to some paths. If saving fails, the file will be written to the configured working directory instead.
* **Enable debug logging in VLC:**
  * GUI: `Tools > Messages`, set verbosity to `2 (debug)` before running VLSub.
  * CLI: `vlc --verbose=2 --file-logging --logfile=vlsub.log` then reproduce the issue. The log will include REST requests (without secrets) and status codes.

## Manual test checklist

* **Windows / macOS / Linux:** install the extension, set an API key, search by hash on a local file, then download and verify the subtitle loads automatically.
* **Slow/unstable network:** lower the timeout to force a retry path, ensure the UI stays responsive, and confirm the rate-limit message appears when expected.
* **Working directory fallback:** temporarily point the working directory to a writable folder and confirm downloads land there when the video directory is read-only.

## Notes

* VLSub now speaks to `api.opensubtitles.com` over the REST API; the legacy XML-RPC flow has been removed.
* Operations yield regularly to VLC to prevent UI freezes during hashing or large downloads. If a request stalls beyond the configured timeout, an explicit error is shown.
