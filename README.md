# DutchEZ 🧀

Make bill splitting a Brie-ze.

## Features

- Add multiple diners
- Add menu items with quantities
- Configure tax and tip percentages
- Share bill calculations via URL links
- Local storage for persistence
- Responsive design

## Local Development

To test the website locally, you have several options:

### Option 1: Python HTTP Server (Recommended)

```bash
python3 server.py
```

Or use the convenience script:

```bash
./start-server.sh
```

The server will start on `http://localhost:8000` and automatically open in your browser.

### Option 2: Python Simple Server

```bash
python3 -m http.server 8000
```

Then open `http://localhost:8000` in your browser.

### Option 3: Node.js http-server

If you have Node.js installed:

```bash
npx http-server -p 8000
```

## Deployment

This repository is set up for GitHub Pages. Simply push your changes to the `main` branch, and GitHub Pages will automatically deploy your site to `https://tlt.github.io`.

Make sure your repository settings have GitHub Pages enabled:
1. Go to Settings → Pages
2. Select the `main` branch as the source
3. Save

## Files

- `index.html` - Main HTML file with all functionality
- `server.py` - Local development server script
- `start-server.sh` - Convenience script to start the server
