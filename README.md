# yt-podcast-rss

Turn a YouTube playlist into a private audio podcast feed you can add to
Overcast, Pocket Casts, or any app that takes a feed URL.

Many shows publish only on YouTube. Podcast apps need an RSS feed with audio
files, and a YouTube playlist feed has neither. This script downloads the
audio with `yt-dlp`, writes `feed.xml`, and leaves serving to your web server.
One Python file, no pip dependencies.

## Requirements

- Python 3.11+
- `yt-dlp`, `ffmpeg`, `curl` on PATH (`brew install yt-dlp ffmpeg`)
- A web server that can serve a folder over HTTPS (Caddy, nginx, ...)

## Setup

```sh
git clone https://github.com/<you>/yt-podcast-rss
cd yt-podcast-rss
mkdir -p ~/.config/yt-podcast-rss
cp config.example.toml ~/.config/yt-podcast-rss/config.toml
$EDITOR ~/.config/yt-podcast-rss/config.toml
./yt-podcast-rss
```

Add `<base_url>/<slug>/feed.xml` to your podcast app (Overcast: + > Add URL).

Run `./yt-podcast-rss my-show` to sync one feed, `-c path.toml` to use another
config. Exit code is 1 if any download failed.

## Serving

Point a web server at `root`. Caddy:

```
podcast.example.com {
    root * /home/me/Podcasts
    file_server
}
```

With `root = "~/Podcasts/<token>"` and `base_url = "https://podcast.example.com/<token>"`,
the feed lands at `https://podcast.example.com/<token>/<slug>/feed.xml`.
Do not enable directory listings. Audio must support HTTP range requests
(Caddy and nginx do by default).

## Scheduling

The script skips episodes it already has, so run it as often as you like.

cron:

```
0 */6 * * * /path/to/yt-podcast-rss
```

macOS launchd (`~/Library/LaunchAgents/com.example.yt-podcast-rss.plist`):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key><string>com.example.yt-podcast-rss</string>
  <key>ProgramArguments</key><array><string>/path/to/yt-podcast-rss</string></array>
  <key>EnvironmentVariables</key>
  <dict><key>PATH</key><string>/opt/homebrew/bin:/usr/bin:/bin</string></dict>
  <key>StartInterval</key><integer>21600</integer>
  <key>RunAtLoad</key><true/>
</dict>
</plist>
```

Load it with `launchctl bootstrap gui/$(id -u) <plist>`. launchd does not
inherit your shell PATH, hence the `PATH` entry.

## Troubleshooting

- `HTTP Error 403` or `Requested format is not available`: update yt-dlp
  first (`brew upgrade yt-dlp`). YouTube changes break old versions often.
- The script tries several YouTube player clients in order (`PLAYER_CLIENTS`)
  and uses the first that works. Which one works changes over time.
- Private and deleted videos are skipped.
- Delete `<root>/<slug>/cover.jpg` to regenerate the cover.

## Legal

Downloading audio from YouTube may violate its Terms of Service. This is for
personal use. Keep the feed URL private (random token in the path) and do not
redistribute other people's content.

## License

MIT
