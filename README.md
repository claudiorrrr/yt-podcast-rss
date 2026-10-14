# yt-podcast-rss

Turn a YouTube playlist into a private podcast feed for Overcast, Pocket Casts
or any app that takes a feed URL.

It downloads the audio with `yt-dlp` and writes `feed.xml`. Your web server
does the serving. One Python file, no pip dependencies.

## Example

The playlist [Real Politik FC](https://www.youtube.com/playlist?list=PL97z6cpmRBbB2VAVGdWS38BgllG0K4idb)
has no podcast feed. Config in `~/.config/yt-podcast-rss/config.toml`:

```toml
root = "~/Podcasts/k9x2-random-token"
base_url = "https://podcast.example.com/k9x2-random-token"

[feeds.real-politik-fc]
playlist = "PL97z6cpmRBbB2VAVGdWS38BgllG0K4idb"
title = "Real Politik FC"
author = "El Orden Mundial x Panenka"
category = "Sports"
description = "Geopolitics and football."
```

Run it:

```
$ yt-podcast-rss
[real-politik-fc] 7 public video(s)
[real-politik-fc] 7 new, 0 failed, 7 episode(s) in feed -> https://podcast.example.com/k9x2-random-token/real-politik-fc/feed.xml
```

In Overcast: + > Add URL, paste that feed URL.

## Setup

Needs Python 3.11+, `yt-dlp`, `ffmpeg`, `curl`.

```sh
brew install yt-dlp ffmpeg
mkdir -p ~/.config/yt-podcast-rss
cp config.example.toml ~/.config/yt-podcast-rss/config.toml
```

Serve `root`'s parent folder over HTTPS. Caddy:

```
podcast.example.com {
    root * /home/me/Podcasts
    file_server
}
```

The random token in the path is your only access control. Keep directory
listings off.

## Keep it updated

Existing episodes are skipped, so run it on a timer:

```
0 */6 * * * /path/to/yt-podcast-rss
```

On macOS use launchd instead. It does not inherit your shell PATH, so set
`PATH` to include `/opt/homebrew/bin` in the plist.

## Usage

```
yt-podcast-rss [slug ...] [-c config.toml]
```

Exit code is 1 if any download failed. Optional feed keys (`language`,
`cover_url`, `strip_suffix`) are in `config.example.toml`.

## Troubleshooting

`HTTP Error 403` or `Requested format is not available`: run
`brew upgrade yt-dlp`. YouTube breaks old versions often.

## Legal

Downloading from YouTube may violate its Terms of Service. Personal use only.
Keep the feed private and do not redistribute other people's content.

MIT license.
