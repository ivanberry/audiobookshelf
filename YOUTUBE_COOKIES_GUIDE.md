# YouTube Cookies Guide

## Why do I need cookies?

Some YouTube videos require authentication (sign-in) to download, or you may encounter "Sign in to confirm you're not a bot" errors. Using cookies allows yt-dlp to access your YouTube session and bypass these restrictions.

## How to export cookies from your browser

### Method 1: Using Browser Extension (Recommended)

1. **For Chrome/Edge**: Install [Get cookies.txt LOCALLY](https://chrome.google.com/webstore/detail/get-cookiestxt-locally/cclelndahbckbenkjhflpdbgdldlbecc)
2. **For Firefox**: Install [cookies.txt](https://addons.mozilla.org/en-US/firefox/addon/cookies-txt/)

3. Go to [YouTube.com](https://www.youtube.com) and sign in to your account
4. Click the extension icon and export cookies for `youtube.com`
5. Save the file as `cookies.txt` in your audiobookshelf directory

### Method 2: Using yt-dlp command (Advanced)

If you have yt-dlp installed locally, you can export cookies directly:

```bash
yt-dlp --cookies-from-browser chrome --cookies cookies.txt https://www.youtube.com/
```

This exports all browser cookies to `cookies.txt`.

## How to use cookies with Docker

1. **Export cookies** using one of the methods above and save as `cookies.txt`

2. **Edit docker-compose.build.yml** and uncomment these lines:

```yaml
volumes:
  # ... other volumes ...
  - ./cookies.txt:/cookies.txt:ro

environment:
  # ... other environment variables ...
  - YT_DLP_COOKIES_PATH=/cookies.txt
```

3. **Restart the container**:

```bash
docker-compose -f docker-compose.build.yml down
docker-compose -f docker-compose.build.yml up -d
```

4. **Test downloading** - your YouTube downloads should now work with authentication

## Important Notes

- **Security**: The cookies.txt file contains your YouTube session. Keep it private and don't share it.
- **File Format**: The cookies file must be in Netscape/Mozilla format (the extensions above export in the correct format)
- **Expiration**: Cookies may expire after some time. If downloads stop working, export fresh cookies.
- **Read-only**: The `:ro` flag in docker-compose mounts the file as read-only for security.

## Troubleshooting

### Error: "Sign in to confirm you're not a bot"
- Make sure cookies.txt is properly exported and mounted
- Verify YT_DLP_COOKIES_PATH environment variable is set correctly
- Try exporting fresh cookies from your browser

### Error: "HTTP Error 400: Bad Request"
- The cookies file may have incorrect newline format
- Re-export cookies using the browser extension
- On Windows, ensure the file has CRLF line endings (`\r\n`)
- On Linux/Mac, ensure the file has LF line endings (`\n`)

### Cookies not being used
- Check the container logs: `docker-compose -f docker-compose.build.yml logs -f`
- Look for: `[YouTubeDownloadManager] Using cookies file: /cookies.txt`
- If you don't see this message, verify the volume mount and environment variable

## Example Configuration

Here's a complete example of the relevant sections in `docker-compose.build.yml`:

```yaml
services:
  audiobookshelf:
    volumes:
      - ./audio/audiobooks:/audiobooks
      - ./audio/podcasts:/podcasts
      - ./audio/metadata:/metadata
      - ./audio/config:/config
      - ./cookies.txt:/cookies.txt:ro  # Add this line

    environment:
      - TZ=Asia/Shanghai
      - YT_DLP_COOKIES_PATH=/cookies.txt  # Add this line
```
