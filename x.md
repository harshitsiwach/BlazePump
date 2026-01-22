# Unity WebGL Integration Guide for X (Twitter)

This guide details the technical requirements and steps to make a Unity WebGL build playable directly within an X (formerly Twitter) feed using **Twitter Player Cards**.

---

## 1. Core Concept

To run a game on X, you must implement a **Twitter Player Card**.
- X does **not** host your game.
- You provide a URL to your hosted WebGL build (e.g., on Vercel).
- You add specific `<meta>` tags to your `index.html`.
- X crawls these tags and renders an `iframe` of your game in the timeline.

---

## 2. Unity Project Configuration

Optimize your build for web embedding before building.

### Player Settings
1.  **Resolution**:
    *   **Square**: `600x600` (Best for mobile feeds)
    *   **Landscape**: `1280x720` (Standard video)
    *   *Note: X respects the aspect ratio defined in your meta tags.*
2.  **Publishing Settings**:
    *   **Compression Format**: `Gzip` or `Brotli` (Crucial for fast loading).
    *   **Decompression Fallback**: Enabled.
3.  **WebGL Template**:
    *   Use **Minimal**.
    *   Disable "Run in Background" if you don't want it consuming resources when scrolled away (optional but recommended).

---

## 3. HTML Configuration (`index.html`)

This is the most critical step. You must add the following `<meta>` tags to the `<head>` section of your **final built `index.html`**.

### Required Meta Tags

```html
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- 1. Card Type: Must be 'player' -->
    <meta name="twitter:card" content="player">
    
    <!-- 2. Site/Creator: Your X handle -->
    <meta name="twitter:site" content="@YourUsername">
    
    <!-- 3. Post Info -->
    <meta name="twitter:title" content="Your Game Title">
    <meta name="twitter:description" content="Click to play this game directly in your feed!">
    
    <!-- 4. Player URL: The HTTPS URL to your index.html -->
    <!-- CRITICAL: Must be the full HTTPS path to where this file is hosted -->
    <meta name="twitter:player" content="https://your-game-url.vercel.app/index.html">
    
    <!-- 5. Player Dimensions: Must match your game's usage -->
    <meta name="twitter:player:width" content="600">
    <meta name="twitter:player:height" content="600">
    
    <!-- 6. Thumbnail Image: Shown before the user clicks play -->
    <!-- Must be HTTPS. Less than 5MB. JPG, PNG, WEBP, or GIF -->
    <meta name="twitter:image" content="https://your-game-url.vercel.app/cover-image.png">
</head>
```

### CSS for Full Frame

Ensure your Unity canvas fills the iframe X creates.

```css
body {
    margin: 0;
    padding: 0;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
    background: #000;
}

#unity-container {
    width: 100%;
    height: 100%;
}

canvas {
    width: 100%;
    height: 100%;
    display: block;
}
```

---

## 4. Hosting Requirements

1.  **HTTPS is Mandatory**: X will not load HTTP content.
2.  **Vercel / GitHub Pages**: Both work excellent.
3.  **Mixed Content**: Ensure your game does not try to load any HTTP resources (APIs, images), or valid browsers will block it.

### Vercel Configuration (`vercel.json`)

If hosting on Vercel, use this config to ensure correct headers and HTTPS enforcement.

```json
{
  "headers": [
    {
      "source": "/:path*",
      "headers": [
        {
          "key": "Content-Security-Policy",
          "value": "frame-ancestors 'self' https://twitter.com https://x.com https://cards-dev.twitter.com;"
        },
        {
          "key": "X-Frame-Options",
          "value": "ALLOW-FROM https://twitter.com"
        }
      ]
    }
  ]
}
```

---

## 5. Validation & Whitelisting

Historically, Twitter required domain whitelisting for Player Cards. While this is often automated now, you should always validate.

1.  **Deploy your game**.
2.  **Copy the URL** (e.g., `https://your-game.vercel.app`).
3.  **Go to the Card Validator**: [https://cards-dev.twitter.com/validator](https://cards-dev.twitter.com/validator)
4.  **Paste your URL** and click "Preview Card".
5.  **Check the Log**:
    *   Look for `twitter:player` URL found.
    *   Look for "Card loaded successfully".

---

## 6. Implementation Checklist for AI Agents

When tasking an AI to build this:

1.  [ ] **Build Unity Project** to WebGL (Standard Output).
2.  [ ] **Inject Meta Tags** into `index.html` (Programmatically or manually).
3.  [ ] **Ensure CSS** sets `canvas` to 100% width/height.
4.  [ ] **Create `vercel.json`** with `frame-ancestors` for `x.com` and `twitter.com`.
5.  [ ] **Deploy** to HTTPS provider.
6.  [ ] **Verify** via Card Validator.