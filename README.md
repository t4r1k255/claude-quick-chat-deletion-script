# Claude – Hızlı Chat Sil

A Tampermonkey userscript that adds a one-click delete button to every conversation in the Claude.ai sidebar — no confirmation dialogs, no page reloads.

## Features

- Hover over any conversation → a red **✕** button appears
- Sends a proper `DELETE` request using the correct auth headers, captured by intercepting the page's own `fetch` calls before they happen (`@run-at document-start`)
- **Ghost entry protection:** deleted conversation IDs are persisted in `localStorage`; even if React re-renders the sidebar after a page refresh, those entries are removed from the DOM immediately
- Redirects to home if you delete the conversation you're currently viewing
- Works across multiple orgs by trying each one until the deletion succeeds

## Installation

1. Install [Tampermonkey](https://www.tampermonkey.net/) for your browser
2. Open the Tampermonkey dashboard → **Create a new script**
3. Paste the contents of [`claude-hizli-sil.user.js`](./claude-hizli-sil.user.js) and save
4. Navigate to [claude.ai](https://claude.ai) — the ✕ buttons will appear on hover in the sidebar

## How It Works

Claude.ai requires specific request headers (`anthropic-client-version`, `x-api-key`, etc.) that aren't accessible from a regular userscript running after page load. The script wraps `window.fetch` at `document-start`, before the page's own JavaScript runs, to silently collect those headers from outgoing API requests. When you click ✕, it fires a `DELETE` to the correct conversation endpoint using those captured headers.

On top of that, a small `localStorage` key (`claude_qs_deleted`) keeps track of every successfully deleted conversation ID. A `MutationObserver` watches the sidebar and removes any entry matching a stored ID as soon as React tries to render it — preventing the "ghost entry" problem where a deleted conversation reappears after a refresh.

## Usage

Hover over any conversation in the left sidebar and click the **✕** that appears on the left edge. That's it.

---

## Disclaimer

> This script is an independent, community-built tool and is **not affiliated with, endorsed by, or associated with Anthropic in any way**. It interacts with Claude.ai's internal API endpoints, which may change at any time without notice and could cause the script to stop working or behave unexpectedly. Use at your own risk. The author takes no responsibility for any unintended deletion of conversations or other side effects.

## License

[MIT](./LICENSE) © [t4r1k255](https://github.com/t4r1k255)
