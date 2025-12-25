# Redgifs Downloader Browser Extension (Chrome, Firefox, Edge, Opera, Brave)
  
A browser extension that adds a download button to RedGifs (redgifs.com) pages to easily download GIFs and short videos for convenient offline viewing.

- Save GIFs and short videos for offline viewing any time
- Protect favorites by downloading before posts are removed
- Build a personal library of clips you can keep
- Avoid losing access if accounts are deleted or content disappears

## Links

- Get it here: https://serp.ly/redgifs-downloader
- Check FAQs: https://github.com/orgs/serpapps/discussions/categories/faq
- Report bugs: https://github.com/serpapps/redgifs-downloader/issues
- Request features: https://github.com/serpapps/redgifs-downloader/issues

## Table of Contents
- [Features](#features)
- [Screenshots](#screenshots)
- [Videos](#videos)
- [Installation Instructions](#installation-instructions)

## Features

- One-click download from RedGifs watch pages and feed grids
- Auto-detect GIFs and videos on the page
- Floating download button overlay
- Privacy-friendly with no tracking or data collection
- Support via the community

## Screenshots

Screenshot coming soon.

## Videos

Video walkthrough coming soon.

## [1.0.0] - 2025-12-24

### Added
- Download button on watch pages and feed grids
- Support for MP4 and HLS sources when available
- Improved detection on single-page navigation

### Supported Pages

| Page Type | Support |
|-----------|---------|
| Watch pages | Yes |
| Explore/feed grid | Yes |
| Embed/ifr | Yes |

### What's New
- Universal page support for RedGifs watch and grid views
- Consistent download experience across page types
- Built with WXT (wxt.dev) for reliable MV3 builds

## Frequently Asked Questions

### Q: Does this work on RedGifs GIFs?
A: Yes, it downloads the MP4/HD source when available.

### Q: Does this work on RedGifs video posts?
A: Yes.

### Q: Does it work on the explore feed?
A: Yes, each card includes a Download button.

## Installation Instructions

Each release has its own specific installation instructions to make it easier to upgrade or roll back.
You can find the installation instructions for the specific version in the release:
- https://github.com/serpapps/redgifs-downloader/releases

### How to Use

1. Visit a RedGifs page where you want to download a GIF or video.
2. Click the extension icon in your browser.
3. If needed, click the video to load it fully.
4. Click "Download" in the popup or on the overlay button.

## Permissions Justifications

### activeTab
Allows the extension to interact with the currently open RedGifs tab when the user activates the extension.

### contextMenus
Adds a right-click option to initiate downloads from supported pages.

### downloads
Saves RedGifs videos and GIFs to the user's device for offline viewing.

### notifications
Notifies the user about download progress, completion, or errors.

### offscreen
Processes HLS streams and video data without interrupting browsing.

### scripting
Injects scripts on RedGifs pages to detect media and attach download UI.

### storage
Stores extension settings and activation state locally.

### tabs
Finds the active tab to coordinate downloads and UI updates.

## About

RedGifs is a platform for sharing short video clips and GIFs. Like many platforms, it does not provide a direct offline download workflow.
This extension adds a simple Download button so you can save content you have access to and keep it for offline viewing.



## Related

- https://github.com/serpapps/redgifs-downloader
- [Redgifs Downloader gist](https://gist.github.com/devinschumacher/d5d197773b7af7fd06dfea31f043bfc8)


---

<details>

<summary>
  Research
</summary>

# How to Download RedGifs Videos: Technical Analysis of Stream Patterns, CDNs, and Download Methods

*A comprehensive research document analyzing RedGifs's video infrastructure, embed patterns, stream formats, and optimal download strategies using modern tools*

**Authors**: SERP Apps  
**Date**: December 2025  
**Version**: 1.0

---

## Abstract

This research document analyzes RedGifs watch and embed pages, the temporary token API flow, and the CDN hosts used to deliver MP4 and GIF assets. It provides practical extraction strategies for API-driven URLs and fallback methods for capturing direct media requests.

## Table of Contents

1. [Introduction](#1-introduction)
2. [RedGifs Video Infrastructure Overview](#2-redgifs-video-infrastructure-overview)
3. [URL Patterns and Detection](#3-url-patterns-and-detection)
4. [Stream Formats and CDN Analysis](#4-stream-formats-and-cdn-analysis)
5. [yt-dlp Implementation Strategies](#5-yt-dlp-implementation-strategies)
6. [FFmpeg Processing Techniques](#6-ffmpeg-processing-techniques)
7. [Alternative Tools and Backup Methods](#7-alternative-tools-and-backup-methods)
8. [RedGifs API Integration](#8-redgifs-api-integration)
9. [Implementation Recommendations](#9-implementation-recommendations)
10. [Troubleshooting and Edge Cases](#10-troubleshooting-and-edge-cases)
11. [Conclusion](#11-conclusion)

---

## 1. Introduction

RedGifs is a short-form media host that serves MP4 clips and GIF variants through a tokenized API. The platform relies on an authentication token to query media metadata and returns direct CDN URLs for playback.

### 1.1 Research Scope

- RedGifs watch and iframe pages
- Temporary token API endpoints
- MP4/GIF URL variants and m4s segments
- CDN hostnames and URL patterns

### 1.2 Methodology

- Inspect token generation and API responses
- Capture direct media requests from network logs
- Validate URLs with yt-dlp and ffprobe

---

## 2. RedGifs Video Infrastructure Overview

### 2.1 Video Hosting Types

- API-driven MP4 and GIF delivery
- fMP4 segments served as .m4s
- Direct MP4 files hosted on thumbnail CDN

### 2.2 CDN Architecture

- api.redgifs.com (token + metadata)
- thumbs2.redgifs.com (MP4/GIF assets)
- media.redgifs.com (segments and alternate encodes)

### 2.3 Video Processing Pipeline

1. User loads watch/iframe page
2. Client requests temporary token from /v2/auth/temporary
3. Client queries /v2/gifs/<id> with Bearer token
4. Client fetches direct MP4/GIF from CDN

### 2.4 Access Control and Authentication

- Bearer token required for API requests
- Tokens expire quickly; refresh when needed
- Referer/Origin headers may be checked

---

## 3. URL Patterns and Detection

### 3.1 Watch Page URL Patterns

```
https://www.redgifs.com/watch/<id>
https://www.redgifs.com/ifr/<id>
```

### 3.2 Embed URL Patterns

```
https://www.redgifs.com/ifr/<id>
```

### 3.3 Direct Media and CDN URL Patterns

```
https://thumbs2.redgifs.com/<Name>.mp4
https://thumbs2.redgifs.com/<Name>.gif
https://media.redgifs.com/<Name>.m4s
```

### 3.4 Regex Patterns for URL Extraction

```regex
redgifs\\.com/(?:watch|ifr)/([^/?#]+)
thumbs2\\.redgifs\\.com/([^/?#]+)\\.(mp4|gif)
```

### 3.5 Command-line URL Extraction

```bash
grep -oE "https?://[^'\" ]+\.(mp4|gif|m4s)" page.html | sort -u
grep -oE "redgifs\\.com/(watch|ifr)/[A-Za-z0-9-]+" page.html | sort -u
```

---

## 4. Stream Formats and CDN Analysis

### 4.1 Stream Formats

| Format | Extension | Notes |
|--------|-----------|-------|
| MP4 (progressive) | .mp4 | Direct file URLs hosted on thumbs CDN |
| GIF | .gif | Animated GIF variants for quick previews |
| fMP4 segments | .m4s | Segmented assets referenced by playlists |

### 4.2 Typical Quality Ladder

| Quality | Typical Resolution | Notes |
|---------|--------------------|-------|
| Low | 360p - 480p | Fast preview streams or mobile variants |
| Medium | 720p | Common default for web playback |
| High | 1080p+ | Available when source uploads are higher quality |

### 4.3 CDN URL Construction and Query Parameters

- API returns multiple URL fields for different encodes
- Filename variants often include size or mobile indicators
- Signed tokens are short-lived

### 4.4 Validation and Inspection Commands

```bash
ffprobe -hide_banner -show_streams "clip.mp4"
ffprobe -hide_banner -show_format "clip.mp4"
```

---

## 5. yt-dlp Implementation Strategies

yt-dlp works well for direct MP4 URLs and can ingest the watch page URL when the extractor resolves the API flow. Use cookies and headers if the API enforces origin checks.

### 5.1 Basic Usage

```bash
yt-dlp [OPTIONS] [--] URL [URL...]
yt-dlp -F "https://example.com/watch/123"
```

### 5.2 Authentication and Cookies

- Use --add-header 'Referer: https://www.redgifs.com/' when direct URLs are gated
- If using the API directly, pass the Bearer token in headers

### 5.3 Format Selection and Output Templates

```bash
yt-dlp -f bestvideo+bestaudio/best "URL"
yt-dlp -o "%(title)s.%(ext)s" "URL"
yt-dlp --download-archive archive.txt "URL"
```

### 5.4 Site-Specific Examples

```bash
yt-dlp "https://www.redgifs.com/watch/<id>"
yt-dlp -f best "https://www.redgifs.com/watch/<id>"
yt-dlp "https://thumbs2.redgifs.com/<Name>.mp4"
```

### 5.5 Batch and Archive Mode

```bash
yt-dlp -a urls.txt --download-archive archive.txt
yt-dlp --no-overwrites --continue "URL"
```

### 5.6 Error Handling Patterns

- Refresh token if API calls return 401
- Use --add-header for Referer/Origin if 403 errors appear

---

## 6. FFmpeg Processing Techniques

FFmpeg is useful for validating MP4 outputs and remuxing any m4s segments referenced by playlists.

### 6.1 Inspect and Validate Streams

```bash
ffprobe -hide_banner -i "https://thumbs2.redgifs.com/<Name>.mp4"
ffmpeg -i "https://thumbs2.redgifs.com/<Name>.mp4" -c copy output.mp4
```

### 6.2 Common Remux and Repair Patterns

```bash
ffmpeg -i "playlist.m3u8" -c copy output.mp4
ffmpeg -i input.mp4 -c copy -movflags +faststart output.mp4
ffprobe -hide_banner -show_streams output.mp4
```

---

## 7. Alternative Tools and Backup Methods

### 7.1 Streamlink

```bash
streamlink "https://www.redgifs.com/watch/<id>" best -o output.mp4
```

### 7.2 aria2c

```bash
aria2c -o clip.mp4 "https://thumbs2.redgifs.com/<Name>.mp4"
```

### 7.3 gallery-dl

```bash
gallery-dl "https://www.redgifs.com/watch/<id>"
```

### 7.4 Browser DevTools

- Look for api.redgifs.com/v2/gifs/<id> responses
- Inspect JSON for mp4/gif URL fields
- Capture direct MP4 request from thumbs2.redgifs.com

---

## 8. RedGifs API Integration

### 8.1 Known Endpoints

- POST https://api.redgifs.com/v2/auth/temporary
- GET https://api.redgifs.com/v2/gifs/<id>

### 8.2 Example Requests

```bash
curl -X POST https://api.redgifs.com/v2/auth/temporary
curl -H 'Authorization: Bearer <token>' https://api.redgifs.com/v2/gifs/<id>
```

### 8.3 Token and Session Handling

- Token responses include an access token and expiry
- Use the token immediately; refresh on 401

---

## 9. Implementation Recommendations

### 9.1 Detection Hierarchy

- Resolve watch/ifr URL to API metadata
- Select highest quality MP4 from JSON response
- Fallback to direct CDN URLs from network logs

### 9.2 Site-Specific Notes

- Prefer MP4 URLs from the API response for stable downloads
- Expose a download button for grid/feeds by reading data attributes

### 9.3 Storage and Naming Strategy

- Use gif or mp4 extension based on selected asset
- Include RedGifs ID in filenames for deduping

---

## 10. Troubleshooting and Edge Cases

- API token expiration mid-download
- Multiple encodes in response; choose best by resolution

---

## 11. Conclusion

RedGifs uses a tokenized API to expose direct MP4/GIF URLs on a CDN. Implementations should request temporary tokens, call the metadata endpoint, and select the best MP4 URL. yt-dlp can handle watch URLs directly, while ffmpeg and aria2c provide stable fallback downloads.

| Tool | Best Use Case | Notes |
|------|---------------|-------|
| yt-dlp | Primary downloader for MP4/HLS | Supports cookies, format selection, retries |
| ffmpeg | Remuxing and validation | Useful for HLS to MP4 conversion |
| streamlink | Live/HLS fallback | Streams to file or pipes into ffmpeg |
| aria2c | Multi-connection HTTP/HLS downloads | Good for large files and retries |
| gallery-dl | Image-first or gallery-heavy sites | Best for gallery or attachment extraction |


---

## Disclaimer and Ethical Use

This document is provided for lawful, personal, or authorized use cases only. Always respect the site terms of service, content creator rights, and applicable laws. If DRM or explicit access controls are present, do not attempt to bypass them; use official downloads or creator-provided access instead.

## Last Updated

December 2025

## Next Review

90 days from last update or when site playback changes are observed.

## Related

- SERP Apps research index (internal)
- SERP extension downloaders (internal)

</details>
