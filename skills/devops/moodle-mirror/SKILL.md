---
name: moodle-mirror
description: Mirror Moodle course/module pages to local folders (e.g., Obsidian vaults), preserving structure, extracting page text/Markdown, and downloading attachments. Use for authenticated Moodle crawling, Cloudflare/SSO handling, and repeatable offline search across course content.
---

# Moodle Mirror

## Overview

Mirror Moodle course pages + attachments to a local directory (HTML/MD/TXT), with retry logic and manual/assisted SSO handling. This is built for Moodle sites that require login and occasionally redirect to external URLs.

## Workflow Decision Tree

1) Default: prefer CDP to reuse a real Chrome session (best for anti-bot)
2) If CDP is unstable for downloads, use `--persistent --channel chrome`
3) If SSO appears, use `--auto-login` to auto-click saved account tiles (MFA may still require manual action)
4) For `mod/url/view.php`, mirror the Moodle wrapper page (`forceview=1`) to avoid external 502s

## Quick Start

1) Start or reuse a logged-in Chrome (CDP mode recommended).
2) Run the crawler with a bounded scope and output folder.
3) Check `_status.md`; remediate any remaining errors with targeted runs.
4) Keep auth state in user profile (default), not inside the Obsidian vault.
5) Default behavior overwrites page files when a URL is (re-)saved. This is intentional to avoid duplicates when re-running to fix errors or fill gaps. If you want versioned snapshots, use a new output folder per run (e.g., append a date).

## Speed + Incremental Updates

- The crawler now extracts links from the *main content* area when possible (avoids pulling in global header/footer links like notifications and unrelated courses).
- Use `--resume` to skip already-saved URLs.
- Use `--update-on-change` so that when a previously-saved page is revisited (typically the course homepage / start URLs), it is only re-saved if the local content changed.
- For faster runs on Moodle, set `--networkidle-wait-ms 0` (default) and keep `--polite-delay-ms` low (even `0` is often fine).

## Prerequisites (Tooling)

- Python with Playwright installed
- Chrome (for CDP or persistent profile)
- Optional: a dedicated Chrome profile for Moodle to avoid impacting daily browsing

If Playwright or browsers are missing, install once:

```powershell
pip install playwright
python -m playwright install
```

Start Chrome with CDP (if not already running):

```powershell
chrome.exe --remote-debugging-port=9222 --user-data-dir=%USERPROFILE%\.codex\state\moodle_profile_cdp
```

## Commands (Primary)

CDP mode (reuses an existing Chrome). Run from the skill folder or replace the script path with an absolute path:

```powershell
python .\scripts\moodle_mirror.py `
  --cdp-url http://127.0.0.1:9222 `
  --start-url "https://moodle.ucl.ac.uk/course/view.php?id=55684" `
  --out-dir "E:\path\to\Obsidian\Course Folder\Moodle Mirror" `
  --format md --rewrite-links `
  --allow-prefix https://moodle.ucl.ac.uk/course/ `
  --allow-prefix https://moodle.ucl.ac.uk/mod/ `
  --allow-prefix https://moodle.ucl.ac.uk/pluginfile.php/ `
  --resume --update-on-change `
  --max-pages 120 --max-downloads 200 `
  --polite-delay-ms 0 --networkidle-wait-ms 0 `
  --block-wait-seconds 180
```

Persistent profile mode (good for downloads if CDP fails):

```powershell
python .\scripts\moodle_mirror.py `
  --persistent --headful --channel chrome `
  --start-url "https://moodle.ucl.ac.uk/course/view.php?id=55684" `
  --out-dir "E:\path\to\Obsidian\Course Folder\Moodle Mirror" `
  --format md --rewrite-links `
  --allow-prefix https://moodle.ucl.ac.uk/course/ `
  --allow-prefix https://moodle.ucl.ac.uk/mod/ `
  --allow-prefix https://moodle.ucl.ac.uk/pluginfile.php/ `
  --resume --update-on-change `
  --max-pages 120 --max-downloads 200 `
  --polite-delay-ms 0 --networkidle-wait-ms 0 `
  --block-wait-seconds 180
```

Status report:

```powershell
python .\scripts\mirror_status.py `
  --index "E:\path\to\...\Moodle Mirror\_index.jsonl" `
  --out   "E:\path\to\...\Moodle Mirror\_status.md"
```

## SSO / Cloudflare Handling

- Use `--auto-login` to auto-click the saved Microsoft account tile.
- Provide a hint with `--auto-login-account "your.name@ucl.ac.uk"` to click the correct account.
- MFA/phone prompts still require manual confirmation; the script will wait for completion.

## Targeted Remediation (Common Issues)

1) External URL returns 502:
   - For `mod/url/view.php`, use `forceview=1` to mirror the internal Moodle page instead of the external site.
2) Resource downloads fail:
   - Use `--persistent --headful --channel chrome` to leverage browser downloads.
3) Cloudflare Verify appears:
   - Wait and complete the challenge; keep `--block-wait-seconds` high enough.

## Ask Before Running (Socratic Prompts)

- Which Moodle domain/course URLs are in scope?
- What folder should the mirror live in (Obsidian path)?
- Do you want full attachments download or a capped number?
- Are you okay with manual MFA if it appears?

## References

- See `references/lessons.md` for troubleshooting and tool choice rationale.
