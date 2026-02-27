---
name: clawhub-ytm-cast
version: 1.0.0
description: "--- name: youtube-music-cast description: Download music from YouTube/YouTube Music and stream to Chromecast via Home Assistant. Complete CLI toolset with web server integration, configuration wizard, and playback controls. version: \"6.0.0\" author: Wobo license: MIT homepage: https://github.com/claw"
---

# YouTube Music Cast

--- name: youtube-music-cast description: Download music from YouTube/YouTube Music and stream to Chromecast via Home Assistant. Complete CLI toolset with web server integration, configuration wizard, and playback controls. version: "6.0.0" author: Wobo license: MIT homepage: https://github.com/clawdbot/skills repository: https://github.com/clawdbot/skills/tree/main/youtube-music-cast user-invocable: true triggers: - play music - cast to chromecast - youtube music - download music - cast music keywords: - youtube - music - chromecast - home-assistant - cast - media-player - streaming - yt-dlp - google-cast - audio - mp3 - free-music category: media requires: bins: - yt-dlp - python3 - curl - jq env: [] config: stateDirs: - ~/.youtube-music-cast metadata: clawdbot: emoji: "🎵"
