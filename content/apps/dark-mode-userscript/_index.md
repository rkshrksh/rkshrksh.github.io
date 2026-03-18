+++
title = "Auto Dark Mode Userscript"
description = "Adds a floating toggle to manually enable a smart dark mode theme per-website."
layout = "app-detail"
hidden = false
weight = 20
links = [
  { name = "GitHub Repository", url = "https://github.com/rkshrksh/dark-mode-userscript" },
  { name = "Install Script", url = "https://raw.githubusercontent.com/rkshrksh/dark-mode-userscript/main/dark-mode-userscript.js" }
]
+++

## Features

- **Opt-in Dark Mode**: Manually enable dark mode per domain; the script remembers your preference.
- **Smart Inversion**: Inverts colors on the page while keeping media elements (images, videos, iframes) in their original, correct colors.
- **Native OS Controls**: Tells the browser to natively use dark dropdowns, input fields, and scrollbars alongside the page content.
- **Floating Toggle**: A simple floating button (`☀️` / `🌙`) at the bottom right of the screen lets you quickly toggle dark mode on or off for the current site.
- **Customizable Position**: Right-click the toggle button to move it to any of the four corners of the screen (stored per-site).
- **Smooth Transitions**: Toggling dark mode animates smoothly rather than harshly flashing the screen.
- **No Flicker**: Applies dark mode immediately on page load (`@run-at document-start`) to prevent bright white flashes.
- **SPA Resilient**: Uses a robust `MutationObserver` alongside history API patches to maintain the extension's functionality across complex Single Page Application navigations.

## How it works

This script injects a global CSS style that applies a CSS filter (`filter: invert(1) hue-rotate(180deg) brightness(0.9) contrast(1.1)`) to the `<html>` element to instantly make light websites dark. It then re-inverts specific media tags (like `img`, `video`, `canvas`, `svg`) so pictures and videos look normal. It stores your per-site preference using `GM_setValue` (or `localStorage` as a fallback) so the dark mode stays enabled natively across page loads and SPA navigations.

## Installation

1. Install a Userscript manager extension for your browser:
   - **Safari**: [Userscripts](https://apps.apple.com/us/app/userscripts/id1463298887) or [Tampermonkey](https://apps.apple.com/us/app/tampermonkey/id1482490089)
   - **Chrome**: [Tampermonkey](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo)
   - **Firefox**: [Tampermonkey](https://addons.mozilla.org/en-US/firefox/addon/tampermonkey/) or [Greasemonkey](https://addons.mozilla.org/en-US/firefox/addon/greasemonkey/)

### Method 1: One-Click Install (Recommended)

After installing a Userscript manager, simply **[click here to install the script](https://raw.githubusercontent.com/rkshrksh/dark-mode-userscript/main/dark-mode-userscript.js)**. Your Userscript manager should automatically open and prompt you to install or update the script.

### Method 2: Manual Install

1. Open the [`dark-mode-userscript.js`](https://github.com/rkshrksh/dark-mode-userscript/blob/main/dark-mode-userscript.js) file in this repository and copy its contents.
2. Open your Userscript manager's dashboard and create a **New Script**.
3. Paste the copied contents, save it, and make sure it's enabled.