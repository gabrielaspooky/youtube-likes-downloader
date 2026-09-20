# YouTube Likes Downloader

A small Python script that downloads the videos of your YouTube playlists to your computer, using [yt-dlp](https://github.com/yt-dlp/yt-dlp). By default it downloads your **Liked videos**, but you can point it at any playlist.

You run it on demand. Videos that were already downloaded are remembered and skipped, so it is safe to run as often as you like.

## How it works

YouTube does not notify anything when you like a video, so the script simply checks your playlist each time you run it and downloads whatever is new:

1. It reads your playlist using the cookies of a browser where you are logged in to YouTube.
2. It skips every video listed in `archive.txt` (the record of previous downloads).
3. It downloads the rest as H.264/AAC `.mp4` files, capped at a maximum resolution (720p by default).

## Requirements

- Python 3.9 or newer
- [yt-dlp](https://github.com/yt-dlp/yt-dlp) (installed with its `default` extras)
- [ffmpeg](https://ffmpeg.org/) (to merge video and audio)
- [Deno](https://deno.com/) 2.3.0 or newer (YouTube requires a JavaScript runtime to solve its download challenges)
- A browser with an active YouTube login (Firefox is the most reliable)

## Setup (one time)

### 1. Get the code

```bash
git clone https://github.com/gabrielaspooky/youtube-likes-downloader.git
cd youtube-likes-downloader
```

Or download the repository as a ZIP and extract it.

### 2. Install Python

Check that it is installed:

```bash
python --version
```

If not, download it from [python.org](https://www.python.org/downloads/). On Windows, tick **"Add python.exe to PATH"** in the installer. On macOS/Linux the command may be `python3`.

### 3. Install yt-dlp

```bash
python -m pip install -U -r requirements.txt
```

This installs `yt-dlp[default]`, which includes the `yt-dlp-ejs` challenge solver scripts.

### 4. Install ffmpeg

| System | Command |
| --- | --- |
| Windows | `winget install ffmpeg` |
| macOS | `brew install ffmpeg` |
| Debian/Ubuntu | `sudo apt install ffmpeg` |

Check it with `ffmpeg -version`.

### 5. Install Deno

| System | Command |
| --- | --- |
| Windows | `winget install DenoLand.Deno` |
| macOS | `brew install deno` |
| Linux | `curl -fsSL https://deno.com/install.sh \| sh` |

Check it with `deno --version`. yt-dlp uses Deno automatically, no extra configuration needed.

> Already have Node.js 22+? You can use it instead: add `"--js-runtimes", "node",` to the command list in `build_command()`.

> After installing anything, **close and reopen your terminal** so the new commands are found.

### 6. Log in to YouTube in your browser

Open Firefox (or the browser set in the script), log in to YouTube with your account, then **close the browser completely** before running the script. Browsers lock their cookie database while running.

## Run

From the project folder:

```bash
python download_likes.py
```

The videos are saved in:

- Windows: `C:\Users\<you>\Videos\YouTube-Likes`
- macOS/Linux: `~/Videos/YouTube-Likes`

Run it again any time: only new videos will be downloaded.

To open the output folder on Windows:

```powershell
explorer $HOME\Videos\YouTube-Likes
```

## Configuration

Edit the constants at the top of `download_likes.py`:

| Setting | Default | Description |
| --- | --- | --- |
| `BROWSER` | `"firefox"` | Browser whose cookies are used (`"chrome"`, `"edge"`, `"brave"`...). |
| `PLAYLISTS` | Liked videos | List of playlist URLs to download. |
| `LIMIT` | `20` | Only check the N most recent videos of each playlist. `None` checks them all. |
| `MAX_HEIGHT` | `720` | Maximum resolution: `360`, `480`, `720`, `1080`... Lower means faster and smaller files. |
| `OUTPUT_DIR` | `~/Videos/YouTube-Likes` | Where the videos and `archive.txt` are saved. |

The first time, consider setting `LIMIT = 3` to test quickly. With `LIMIT = None` the whole playlist is downloaded, which can take a long time and a lot of disk space.

### Use a different playlist

Create a playlist on YouTube (for example "To download"), add the videos you want, and put its URL in `PLAYLISTS`:

```python
PLAYLISTS = [
    "https://www.youtube.com/playlist?list=PLxxxxxxxxxxxxxxxx",
    # "https://www.youtube.com/playlist?list=LL",  # uncomment to include your Liked videos
]
```

Public, unlisted and private playlists all work, since the script uses your browser session. Removing a video from the playlist does not delete it from your computer.

## Re-downloading a video

Downloaded videos are recorded in `archive.txt` (inside the output folder), one line each:

```
youtube dQw4w9WgXcQ
```

- To download one video again, delete its line from `archive.txt` (and the old file, if it is still there).
- To download everything again, delete `archive.txt`.
- If you change `OUTPUT_DIR`, move `archive.txt` to the new folder, or everything will be downloaded again.

## Troubleshooting

**`Could not copy Chrome cookie database`**
The browser is still running or its cookies are encrypted. Close it completely (on Windows: `taskkill /F /IM chrome.exe`) or, better, use Firefox.

**`n challenge solving failed` / `The page needs to be reloaded`**
yt-dlp cannot solve YouTube's JavaScript challenges. Make sure Deno is installed (`deno --version`) and reinstall yt-dlp with its extras:

```bash
python -m pip install -U "yt-dlp[default]"
```

**No module named yt_dlp**
yt-dlp is not installed for this Python. Run `python -m pip install -U -r requirements.txt` (use the same `python` you run the script with).

**Videos play audio but no picture**
The player lacks an AV1 decoder. The script already prefers H.264, but if a video only exists in another codec, use [VLC](https://www.videolan.org/) or [mpv](https://mpv.io/).

**Downloads suddenly stop working**
YouTube changes often. Update yt-dlp:

```bash
python -m pip install -U "yt-dlp[default]"
```

**A few errors about private or deleted videos**
Normal. The script skips them and continues with the rest.

## Security note

The script reads your browser cookies, which give access to your Google session. They are only used locally by yt-dlp. If you ever export a `cookies.txt` file, treat it like a password and **never commit it** (it is already in `.gitignore`).

## Disclaimer

This project is meant for personal use with content you have access to. Downloading videos may violate YouTube's Terms of Service or copyright law depending on your country and the content. You are responsible for how you use it.
