# PWA Install Prompt Implementation Guide

## Overview

This guide describes how to make an HTML web application installable as a desktop app in Chrome (and other Chromium-based browsers). When properly implemented, users will be prompted to install the app to their system, which adds it to their desktop/start menu with the option to pin to the taskbar.

## Requirements

For the browser to offer installation, the app must meet these criteria:

1. Served over HTTPS (Netlify handles this automatically)
2. Has a valid Web App Manifest
3. Has a registered Service Worker
4. Meets browser engagement heuristics (user has interacted with the site)

## File Structure

```
/
├── index.html
├── manifest.json
├── sw.js
├── icon-192.png
├── icon-512.png
└── (your other app files)
```

## Implementation

### 1. Create the Web App Manifest

Create `manifest.json` in your project root:

```json
{
  "name": "Your Application Name",
  "short_name": "AppName",
  "description": "Brief description of your application",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "orientation": "any",
  "icons": [
    {
      "src": "/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    }
  ]
}
```

**Important fields:**

- `display: "standalone"` — Makes the installed app run in its own window without browser UI
- `start_url` — The page that opens when the app launches
- `icons` — Must include at least 192x192 and 512x512 PNG icons

### 2. Create the Service Worker

Create `sw.js` in your project root. A minimal service worker is sufficient to enable installation:

```javascript
// sw.js - Minimal service worker to enable PWA installation

const CACHE_NAME = 'app-cache-v1';

// Install event - cache core assets (optional but recommended)
self.addEventListener('install', (event) => {
  self.skipWaiting();
});

// Activate event - clean up old caches
self.addEventListener('activate', (event) => {
  event.waitUntil(clients.claim());
});

// Fetch event - required for PWA, can be minimal
self.addEventListener('fetch', (event) => {
  // For a basic implementation, just let requests pass through
  // Add caching logic here if offline support is desired
});
```

### 3. Update Your HTML

Add these elements to the `<head>` of your `index.html`:

```html
<!-- Web App Manifest -->
<link rel="manifest" href="/manifest.json">

<!-- Theme color for browser chrome -->
<meta name="theme-color" content="#000000">

<!-- iOS Safari support (optional but recommended) -->
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="default">
<meta name="apple-mobile-web-app-title" content="AppName">
<link rel="apple-touch-icon" href="/icon-192.png">
```

### 4. Add the Install Prompt Script

Add this script to your HTML (before the closing `</body>` tag or in a separate JS file):

```javascript
// PWA Install Prompt Handler

let deferredPrompt;
let installButton;

// Wait for DOM to be ready
document.addEventListener('DOMContentLoaded', () => {
  // Create install button (or select your existing button)
  installButton = document.getElementById('installButton');
  
  // If no button exists in HTML, create one dynamically
  if (!installButton) {
    installButton = document.createElement('button');
    installButton.id = 'installButton';
    installButton.textContent = 'Install App';
    installButton.style.cssText = `
      display: none;
      position: fixed;
      bottom: 20px;
      right: 20px;
      padding: 12px 24px;
      background: #007bff;
      color: white;
      border: none;
      border-radius: 8px;
      font-size: 16px;
      cursor: pointer;
      box-shadow: 0 2px 10px rgba(0,0,0,0.2);
      z-index: 10000;
    `;
    document.body.appendChild(installButton);
  }

  // Handle install button click
  installButton.addEventListener('click', async () => {
    if (!deferredPrompt) return;
    
    // Show the browser's install prompt
    deferredPrompt.prompt();
    
    // Wait for user response
    const { outcome } = await deferredPrompt.userChoice;
    console.log(`User ${outcome === 'accepted' ? 'accepted' : 'dismissed'} the install prompt`);
    
    // Clear the deferred prompt
    deferredPrompt = null;
    installButton.style.display = 'none';
  });
});

// Capture the install prompt event
window.addEventListener('beforeinstallprompt', (e) => {
  // Prevent the default mini-infobar from appearing
  e.preventDefault();
  
  // Save the event for later use
  deferredPrompt = e;
  
  // Show your custom install button
  if (installButton) {
    installButton.style.display = 'block';
  }
  
  console.log('Install prompt captured and ready');
});

// Detect successful installation
window.addEventListener('appinstalled', () => {
  console.log('App was installed successfully');
  deferredPrompt = null;
  if (installButton) {
    installButton.style.display = 'none';
  }
});

// Register the service worker
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then((registration) => {
      console.log('Service Worker registered:', registration.scope);
    })
    .catch((error) => {
      console.error('Service Worker registration failed:', error);
    });
}
```

### 5. Create App Icons

You need PNG icons at minimum two sizes:

- `icon-192.png` — 192×192 pixels
- `icon-512.png` — 512×512 pixels

**Tips:**

- Use square images with transparent or solid backgrounds
- Keep important content away from edges (for maskable icon cropping)
- Test with both light and dark system themes

## Alternative: Auto-Prompt on Engagement

If you want to automatically show the install prompt after the user has engaged with the app (rather than showing a persistent button), replace the install button logic with:

```javascript
let deferredPrompt;
let hasPrompted = false;

window.addEventListener('beforeinstallprompt', (e) => {
  e.preventDefault();
  deferredPrompt = e;
  
  // Auto-prompt after user has been on page for 30 seconds
  // and has interacted with the page
  if (!hasPrompted) {
    setTimeout(() => {
      if (deferredPrompt && document.hasFocus()) {
        showInstallPrompt();
      }
    }, 30000);
  }
});

// Also prompt after significant user interaction
document.addEventListener('click', () => {
  if (deferredPrompt && !hasPrompted) {
    // Wait a moment so it doesn't feel abrupt
    setTimeout(showInstallPrompt, 2000);
  }
}, { once: true });

async function showInstallPrompt() {
  if (!deferredPrompt || hasPrompted) return;
  hasPrompted = true;
  
  deferredPrompt.prompt();
  const { outcome } = await deferredPrompt.userChoice;
  console.log(`Install prompt outcome: ${outcome}`);
  deferredPrompt = null;
}
```

## Netlify Configuration

No special Netlify configuration is required. Ensure your `manifest.json` and `sw.js` are in your publish directory. Optionally, add a `_headers` file for caching:

```
/sw.js
  Cache-Control: no-cache

/manifest.json
  Content-Type: application/manifest+json
```

## Testing

1. **Chrome DevTools:** Open DevTools → Application tab → Manifest section to verify your manifest is valid
2. **Lighthouse:** Run a Lighthouse audit and check the PWA section
3. **Installation:** The install option appears in Chrome's address bar (desktop icon) or menu when criteria are met

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Install prompt never appears | Check DevTools → Application → Manifest for errors. Ensure service worker is registered. |
| "No matching service worker detected" | Verify `sw.js` is in the root and accessible |
| Icons not showing | Check paths are absolute (`/icon-192.png` not `icon-192.png`) |
| Works locally but not on Netlify | Clear browser cache and service worker, redeploy |

## Browser Support

- **Full support:** Chrome, Edge, Opera, Samsung Internet
- **Partial support:** Firefox (Android only)
- **No support:** Safari (uses different mechanism for "Add to Home Screen" on iOS)

## Notes

- The `beforeinstallprompt` event may not fire on first visit due to browser engagement heuristics
- Users can always manually install via Chrome menu → "Install [App Name]..." even without the prompt
- Once installed, the app runs in its own window and appears in the system's app list
