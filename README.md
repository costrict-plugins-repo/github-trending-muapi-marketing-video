# MuAPI Video Marketing Plugin for Claude Code

Create AI-powered marketing videos directly from **Claude Code** using [MuAPI](https://muapi.ai)'s generative media APIs.

```
Brief → Storyboard → Images → Videos → Music → Delivery
```

---

## What You Can Build

| Request | Pipeline |
|---------|----------|
| "Create a 30s TikTok for our coffee brand" | Storyboard → images → clips → music |
| "Make a product reveal video" | Image upload → image-to-video → effects → music |
| "Animate this logo into a video ad" | Upload → image-to-video with zoom → music |
| "Add background music to my video" | Suno music gen → deliver audio URL |
| "Apply cinematic film noir effect to my clip" | Video effects API |
| "Lipsync this spokesperson to my voiceover" | Lipsync API |

---

## Installation

### Prerequisites

- **Claude Code** [installed and authenticated](https://code.claude.com/docs/en/quickstart)
- **Node.js v18+** installed
- **MuAPI API key** from [https://muapi.ai/dashboard](https://muapi.ai/dashboard)

### Install from Marketplace

```bash
claude plugin install muapi-video-marketing
```

You'll be prompted for your MuAPI API key during setup.

### Install for Local Development

```bash
claude --plugin-dir ./muapi-marketing-video-plugin
```

### Test It Works

Open Claude Code and ask:

```
Create a single test image: a sleek dark background with lime-green neon text "Hello World", photorealistic, 8k
```

Claude will call `muapi_generate_image` and return an image URL. If you get a URL back, you're set up correctly.

### Use the Skill

```
/muapi-video-marketing:muapi-video-marketing
```

Or simply describe your video idea — Claude will automatically use the skill based on context.

---

## Plugin Structure

```
muapi-marketing-video-plugin/
├── .claude-plugin/
│   └── plugin.json               # Plugin manifest & MCP config
├── skills/
│   └── muapi-video-marketing/
│       └── SKILL.md              # Pipeline logic & guidance
├── hooks/
│   └── hooks.json                # Auto-installs deps on session start
├── mcp-server/
│   ├── src/
│   │   └── index.ts              # MCP server with 10 tools
│   ├── package.json
│   └── tsconfig.json
├── scripts/
│   └── setup.sh                  # Manual setup (alternative)
└── README.md                     # This file
```

---

## MCP Tools Reference

| Tool | Description |
|------|-------------|
| `muapi_generate_image` | Text → image via Flux Dev (12B params) |
| `muapi_image_to_video` | Static image → animated video clip |
| `muapi_generate_video` | Text → video (no source image) |
| `muapi_create_music` | Generate background music via Suno V5 |
| `muapi_lipsync` | Sync lips to audio track |
| `muapi_video_effects` | Apply Wan AI cinematic effects |
| `muapi_upload_file` | Upload local file → CDN URL |
| `muapi_poll_result` | Check async task status |
| `muapi_run_workflow` | Execute a saved MuAPI workflow |
| `muapi_list_workflows` | Discover available workflows |

---

## Pipeline Deep Dive

### Standard Marketing Video

```
1. Claude generates storyboard (3-8 scenes as JSON)
2. muapi_generate_image x N   → scene images (Flux Dev, parallel)
3. muapi_image_to_video x N   → scene clips (Kling Pro, sequential)
4. muapi_create_music x 1     → background track (Suno V5)
5. Claude delivers: clip URLs + music URL + assembly guide
```

### Asset-First (User provides images/video)

```
1. muapi_upload_file           → CDN URL for each local asset
2. muapi_image_to_video        → animate provided images
3. muapi_create_music          → background track
```

### Spokesperson Video

```
1. muapi_generate_image        → scene with person
2. muapi_image_to_video        → animate
3. User provides voiceover
4. muapi_lipsync               → sync lips to voiceover
```

---

## Model Guide

### Image Models

| Model ID | Quality | Speed | Best For |
|----------|---------|-------|----------|
| `flux-dev` | High | Medium | Default, high quality |
| `flux-schnell` | Good | Fast | Drafts, prototyping |
| `midjourney-v7` | High | Slow | Premium aesthetics |

### Video Models

| Model ID | Quality | Speed | Best For |
|----------|---------|-------|----------|
| `kling-pro` | High | Slow | Product shots, realism |
| `kling-standard` | Good | Medium | General use |
| `minimax-pro` | High | Medium | Dynamic motion, social |
| `runway-gen3` | Good | Fast | Artistic, creative |
| `wan-2.1` | High | Slow | High fidelity, speech |

### Music (Suno)

Valid model values: `"V5"` (default), `"V4_5"`, `"V4"`, `"V3_5"`

The `style` parameter is **required** by the API.

---

## Aspect Ratio Guide

| Format | Ratio | Size | Platform |
|--------|-------|------|----------|
| Landscape | 16:9 | 1344x768 | YouTube, TV ads |
| Portrait | 9:16 | 768x1344 | TikTok, Reels, Stories |
| Square | 1:1 | 1024x1024 | Instagram feed |

---

## Assembly

After Claude delivers clip and music URLs, use any video editor to assemble:

- **CapCut** (free, mobile/desktop) — import clips, arrange timeline, add music
- **DaVinci Resolve** (free, desktop) — professional grade
- **FFmpeg** (command line):

```bash
# Merge clips
ffmpeg -f concat -safe 0 -i clips.txt -c copy merged.mp4

# Add music
ffmpeg -i merged.mp4 -i music.mp3 \
  -filter_complex "[1:a]volume=0.3[music];[0:a][music]amix=inputs=2:duration=first" \
  -c:v copy final.mp4
```

---

## Pricing (Approximate MuAPI Credits)

| Operation | Credits |
|-----------|---------|
| Image generation (Flux Dev) | ~2-5 |
| Image-to-video (Kling Pro, 5s) | ~15-25 |
| Music (Suno, 30s) | ~5-10 |
| Video effects | ~10-20 |

A typical 30s TikTok (6 scenes): ~100-200 credits.
Top up at [https://muapi.ai/dashboard](https://muapi.ai/dashboard).

---

## Troubleshooting

**MCP server not starting**
- Ensure Node.js v18+ is installed
- Check plugin errors: run Claude Code with `claude --debug`

**401 Unauthorized**
- Re-enter your API key: disable and re-enable the plugin

**404 Not Found**
- Check [https://muapi.ai/docs](https://muapi.ai/docs) for current endpoints

**422 on music generation**
- Ensure you pass a `style` field (e.g. `"electronic, no vocals"`)

**Task failed / timeout**
- Retry with a simpler prompt
- Use `wait_for_result: false` and poll with `muapi_poll_result`

---

## Support

- MuAPI Docs: [https://muapi.ai/docs/introduction](https://muapi.ai/docs/introduction)
- MuAPI Discord: [https://discord.gg/zpnuBRXhKg](https://discord.gg/zpnuBRXhKg)
- MuAPI Support: support@vadoo.tv
