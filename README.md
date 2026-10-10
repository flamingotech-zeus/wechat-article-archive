# WeChat Article Archive

[English](README.md) | [简体中文](README.zh-CN.md)

> Pull a WeChat Official Account article's full text and its hotlink-protected images into a local archive.

## What this is

This is not a bookmark saver and not a "print to PDF" recipe. Both of those lose the images: WeChat serves article images through a referer-checked CDN, so a plain fetch gets you text with broken placeholders.

The skill works through that: fetch the text one way, extract the real image URLs another way, then download each one with the right referer attached.

## Why we open-sourced it

We archive articles constantly — competitor teardowns, industry analysis, anything that informs a decision. Saving the link is not enough, because links rot and articles get edited or removed.

We use it in the company for exactly this kind of routine research archiving. It contains no proprietary method, so sharing it is straightforward.

## What it does

- **Full text extraction.** Title, publishing account, publish time, body text, subheadings and inline data are extracted in one pass.
- **Real image URLs.** The browser-based step recovers the actual image addresses that the plain HTML does not expose.
- **Referer-correct download.** Each image is fetched with the referer WeChat's CDN expects, which is what makes the download succeed.
- **Per-image review.** Images are opened and read one by one, so a silent failure to download does not pass as success.
- **Local archive output.** Text and images end up in a folder structure you can search, back up and keep.

## Requirements

Python 3 · a usable browser automation environment

## Installation

```bash
git clone https://github.com/<your-username>/wechat-article-archive.git
```

Then place the folder into your agent skills directory:

| OS | Path |
|---|---|
| Windows | `C:\Users\<you>\.workbuddy\skills\wechat-article-archive\` |
| macOS / Linux | `~/.workbuddy/skills/wechat-article-archive/` |

There is no build step. Restart the session and it takes effect.

## How to use

Paste a link and say what you want:

- "Read this article and tell me what's useful"
- "Archive this article with its images"
- "Save this article into the knowledge base"

The skill fetches the text, recovers the image addresses, downloads with the correct referer, and then summarises if you asked for that.

## Files

```
└── SKILL.md
```

## Notes and limits

1. **Image links expire in practice.** Archiving now is the only reliable way to keep the images; re-fetching an old URL later frequently fails.
2. **The browser step is required for images.** There is no text-only shortcut around it if you want the pictures.
3. **Article structure varies.** Some accounts use heavy custom formatting. Extraction handles the common cases and is explicit about what it does not.
4. **Respect the source.** Archived content keeps its original attribution; this tool does not strip credit.

## License

MIT
