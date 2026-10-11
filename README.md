# Androcies-DLNA-Server
A lightweight Windows DLNA/UPnP media server supporting video, audio, playlists, and NVIDIA GPU-accelerated live desktop streaming. Developed by Androcies with OpenAI Codex.
> **Recommended player: [VLC media player](https://www.videolan.org/vlc/).** VLC is a good choice for browsing and playing media from the server on Windows PCs, Android/Google TV and Amazon Fire TV devices. Availability and menu names vary by device.

## Features

- **DLNA/UPnP discovery:** browse a shared media library from compatible devices on your home network.
- **Video and audio sharing:** serve supported files directly over HTTP, including byte-range requests for seeking.
- **Playlists:** read `.m3u`, `.m3u8` and `.pls` files and present their entries through DLNA browsing.
- **Multiple media locations:** configure shared folders and playlist files using a simple text file.
- **Optional live Windows desktop streaming:** H.264 video streamed over HTTP, with automatic encoder selection where supported.
- **Configurable:** media paths and streaming options are kept outside the program in editable files.
- **Local logging:** server activity is recorded in `DLNA_Service.log`, with the current log replaced after its 24-hour logging period.

The media server does **not** automatically transcode ordinary video or audio library files. Whether a particular file plays depends on its codecs and the receiving device. **VLC is recommended** because it supports a broad range of formats.

## Getting started (Windows)

1. **Obtain the application.** When a packaged Windows version is published, download it from the repository's [Releases page](https://github.com/Androcies/Androcies-DLNA-Server/releases). If running from source, use the project's Python setup and `DLNA_Service.py` instead.
2. **Extract the application into a writable folder.** Keep supplied configuration files and folders alongside `DLNA_Service.exe` (or `DLNA_Service.py`). You can choose your own installation location.
3. **Configure your shared media** by editing `MediaPaths\Media.txt`, as shown below.
4. **Start the server** by running `DLNA_Service.exe`. For a configured source installation, run `python -u DLNA_Service.py` from the application directory.
5. **Allow private-network access** when prompted by Windows Firewall. Third-party firewalls may need their own permission for the program.
6. **Open VLC** on another device connected to the same local network and browse **Local Network → Universal Plug'n'Play (UPnP)** (the menu name may vary). Select **Androcies-DLNA-Server** or the server name shown by your installation.

Leave the server running while watching or listening. Press **Ctrl+C** in its console window to stop it.

**Note:** This repository may not yet have a downloadable EXE release or all source files. Check the Releases page and repository contents for what is currently published.

## Configure your media library

Edit the following file beside the application:

```text
MediaPaths\Media.txt
```

Enter **one path per line**. A path can point to a folder containing shared media, a folder containing playlists, or an individual playlist file. For example:

```text
D:\Videos
D:\Music
D:\Playlists
D:\Playlists\Favourites.m3u
```

Use paths that exist **on the Windows computer running the server**. These are examples, not required folders. Blank lines are ignored.

### Supported file extensions

| Type | Extensions |
| --- | --- |
| Video | `.mp4`, `.mkv`, `.avi`, `.mov`, `.wmv`, `.webm`, `.mpg`, `.mpeg`, `.m4v`, `.ts`, `.m2ts`, `.flv`, `.3gp` |
| Audio | `.mp3`, `.flac`, `.wav`, `.aac`, `.m4a`, `.ogg`, `.wma`, `.opus`, `.aiff` |
| Playlists | `.m3u`, `.m3u8`, `.pls` |

Playlist entries can refer to media using relative or absolute Windows paths, but **the referenced media must be inside a directory you have explicitly shared in `Media.txt`**. A playlist alone does not grant access to other files on your computer.

Playlists appear in a **PLAYLISTS** area of the DLNA library where supported. The exact browse view and whether a client automatically advances through playlist entries depend on the client application.

## Recommended playback: VLC

### Movies, music and playlists

- **VLC for Windows:** Open **View → Playlist**, then look under **Local Network → Universal Plug'n'Play**.
- **VLC for Android TV / Fire TV:** Open **Browsing / Local Network** and select the detected UPnP server. Screen labels vary by VLC version.
- Browse the available folders and select a video, audio file or playlist entry.

If the server does not appear, verify that both devices are on the same home network and check the firewall notes below.

### Optional live desktop video

Desktop streaming is **disabled by default** for privacy. To enable it, edit `Settings.ini` in the application folder:

```ini
[DesktopStreaming]
Enabled = True
Encoder = Auto
```

A compatible **FFmpeg** executable may also be required, depending on the distribution. If FFmpeg is not included and the application cannot find it, configure `FFmpegPath` in `Settings.ini` to point to a compatible local `ffmpeg.exe`. Hardware encoding is selected automatically when available; software encoding may be used as a fallback.

Start or restart the server, and note the **LAN IP address** displayed in its console. In VLC, choose **Open Network Stream** and enter one of the following, replacing `<SERVER-IP>` with that address:

```text
http://<SERVER-IP>:8200/live/desktop
http://<SERVER-IP>:8200/live/high-quality
http://<SERVER-IP>:8200/live/low-latency
http://<SERVER-IP>:8200/live/gaming
```

For example, if the server displays `192.168.1.50`, use `http://192.168.1.50:8200/live/desktop`. The IP address may change when your router assigns a new one. The port shown above is the default; use the configured port if you have changed it.

To prioritise clear desktop text (for example, Microsoft Word), a suitable *starting point* is:

```ini
[DesktopProfile]
Resolution = 1920x1080
FPS = 30
BitrateMbps = 6
```

Actual detail is limited by the captured display's resolution; requesting 4K does not create genuine 4K detail from a 1080p desktop. If playback is unreliable, reduce the bitrate and/or resolution.

**Important desktop-streaming limitations:**

- The live stream currently carries **desktop video only**, not Windows system audio.
- VLC may buffer the video by **several seconds**, even with the low-latency profile. It is suitable for passive viewing but **not recommended for gaming or instant-response remote desktop work**. For interactive use, consider Steam Link or Sunshine/Moonlight instead.
- The desktop stream is not a remote-control feature: it does not transmit keyboard or mouse input back to Windows.
- **Anyone on your local network who can reach an enabled live-stream URL may be able to view your screen.** Enable it only on a trusted private network; do not forward the server's port to the internet.

## Network and firewall notes

- **TCP port 8200** (default): HTTP media and live desktop streaming.
- **UDP port 1900:** UPnP/SSDP discovery.
- Windows Firewall may request permission for the packaged executable. Allow access on **Private** networks only as appropriate for your home setup.
- Third-party firewall software, such as **GlassWire**, can block the server even when Windows Firewall allows it; grant access separately if necessary.
- Devices must be able to communicate on the local network; guest Wi-Fi or client-isolation settings may prevent discovery or playback.

## Files and privacy

| File or folder | Purpose |
| --- | --- |
| `Settings.ini` | Server and optional desktop-streaming settings |
| `MediaPaths\Media.txt` | User-defined shared folders and playlist locations |
| `VideoTracker\playback.db` | Local playback-tracking data |
| `DLNA_Service.log` | Diagnostic log, periodically overwritten |

**Before publishing your own copy of the project, remove private paths, network details, playback databases and log files.** Use example configuration files for public repositories rather than committing machine-specific settings.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| VLC cannot find the server | Both devices on the same LAN; UPnP browsing; UDP 1900; Windows and third-party firewall permissions; router client isolation |
| VLC sees the server but cannot play media | File codec compatibility; file permissions; TCP 8200; try VLC on a Windows PC as a comparison |
| A playlist entry does not appear | Confirm its media file is inside a folder explicitly shared in `Media.txt`; verify the referenced path exists |
| Live desktop URL does not play | Desktop streaming enabled; correct current server IP and complete `/live/...` path; FFmpeg configured; lower bitrate/resolution if required |
| Live desktop playback is delayed | A delay of seconds is possible with VLC and HTTP MPEG-TS; this is a playback/streaming limitation, not necessarily a slow network |

## Project and attribution

**Androcies-DLNA-Server** was developed by **Androcies** with assistance from **OpenAI Codex**.

- **Project:** [github.com/Androcies/Androcies-DLNA-Server](https://github.com/Androcies/Androcies-DLNA-Server)
- **Website:** [www.androcies.com](https://www.androcies.com)

License and redistribution terms are governed by the license, if any, published with the repository. No license is implied by the availability of this README.
