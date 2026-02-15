# My Ad-Block Filters

![Filter Lists](https://img.shields.io/badge/filter_lists-2_platforms-blue)
![Sites Covered](https://img.shields.io/badge/sites-YouTube%20%7C%20LinkedIn%20%7C%20M365%20%7C%20General-green)
![Auto Build](https://img.shields.io/github/actions/workflow/status/YOUR_USERNAME/my-adblock-filters/build.yml?label=build)
![Last Commit](https://img.shields.io/github/last-commit/YOUR_USERNAME/my-adblock-filters)

Custom ad-blocking and cleanup filter rules maintained for two platforms: **uBlock Origin** (desktop browsers — Firefox/Brave) and **AdGuard** (mobile — iOS/Android). Each platform has its own optimized filter set, with per-site rule files that are combined via barrel files and an automated build pipeline.

## Quick Setup

### uBlock Origin (Desktop)

1. Open uBlock Origin dashboard (click the extension icon → gear icon)
2. Go to **Filter lists** tab
3. Scroll to the bottom → **Import...** → paste this URL:

```
https://raw.githubusercontent.com/YOUR_USERNAME/my-adblock-filters/main/ublock/all.txt
```

4. Click **Apply changes**

### AdGuard (Mobile)

1. Open AdGuard → **Settings** → **Content Blocking** (or **Safari Protection** on iOS)
2. Go to **Custom filters** (or **User rules**)
3. Tap **Add a custom filter** → paste this URL:

```
https://raw.githubusercontent.com/YOUR_USERNAME/my-adblock-filters/main/adguard/all-flat.txt
```

4. Enable the filter and save

> **Note:** The AdGuard version uses `all-flat.txt` (a concatenated single file) instead of `all.txt` because `!#include` directives may not work reliably in AdGuard's user-imported custom filter lists.

## What's Included

| File | Platform | Description |
|------|----------|-------------|
| `youtube.txt` | Both | Shorts removal, home page cleanup, player de-cluttering, sidebar filtering, search result cleanup, subscription/channel page improvements, navigation cleanup, tracking parameter removal |
| `linkedin.txt` | Both | Sponsored/promoted post removal, reaction-based post filtering, Premium upsell blocking, AI feature/Copilot promo removal, sidebar cleanup, messaging widget removal, job board ad filtering, cookie consent, tracking parameter removal |
| `microsoft365.txt` | Both | Outlook Web ad removal, Teams upgrade/promo banners, SharePoint feedback prompts, OneDrive storage upsells, Copilot promotion removal, M365 portal cleanup, Microsoft telemetry blocking, Bing/MSN ad network blocking |
| `general.txt` | Both | Cross-site UTM/tracking parameter stripping, newsletter popup removal, push notification prompts, app download banners, third-party tracker blocking (analytics, pixels) |

## Platform Differences

The AdGuard versions are adapted from the uBlock Origin rules with the following changes:

- **`:matches-attr()` rules omitted** — AdGuard doesn't support this selector. Affects: already-watched video detection (progress bar percentage matching), low view count filtering
- **`:matches-path()` rules omitted** — AdGuard doesn't support path-aware selectors. Affects: path-conditional Shorts removal (e.g., preserving Shorts in watch history)
- **`:remove-attr()` rules omitted** — AdGuard doesn't support attribute removal. Affects: fullscreen scroll fix, seasonal logo toggle
- **`:upward(n)` rewritten** — uBO's `:upward()` selector is replaced with `:has()` chains from the parent element in AdGuard rules
- **`:xpath()` rules omitted** — AdGuard doesn't support XPath selectors. Affects: some LinkedIn sponsored post detection using ancestor traversal
- **`$xhr` → `$xmlhttprequest`** — AdGuard uses the full option name for XMLHttpRequest filtering

All basic cosmetic filters (`##`), `:has()`, `:has-text()`, `$removeparam`, `$third-party`, and CSS injection (`#$#`) rules are compatible across both platforms.

## Adding New Sites

1. Create `sitename.txt` in **both** `ublock/` and `adguard/` folders
   - Start with the standard header: `! Title:`, `! Description:`, `! Last modified:`
   - Add uBO-specific syntax in the uBlock version
   - Translate to AdGuard-compatible syntax in the AdGuard version (see syntax table below)
2. Add `!#include sitename.txt` to **both** `ublock/all.txt` and `adguard/all.txt` barrel files
3. Push to `main` — the GitHub Action automatically rebuilds the flat concatenated files

### Syntax Translation Reference

| Feature | uBlock Origin | AdGuard | Action |
|---------|--------------|---------|--------|
| Basic cosmetic | `##selector` | `##selector` | Copy as-is |
| `:has()` | `##el:has(child)` | `##el:has(child)` | Copy as-is |
| `:has-text()` | `##el:has-text(/regex/)` | `##el:has-text(/regex/)` | Copy as-is |
| `:upward(n)` | `##el:upward(3)` | *Not supported* | Restructure with `:has()` or omit |
| `:upward(sel)` | `##el:upward(div.parent)` | *Not supported* | Use `div.parent:has(el)` |
| `:matches-attr()` | `##el:matches-attr(...)` | *Not supported* | Omit |
| `:remove()` | `##el:remove()` | `##el { remove: true; }` | Translate syntax |
| `:remove-attr()` | `##el:remove-attr(x)` | *Not supported* | Omit |
| `:style()` | `##el:style(css)` | `#$#el { css }` | Translate |
| `$xhr` | `$xhr` | `$xmlhttprequest` | Rename |
| `$removeparam` | `$removeparam=x` | `$removeparam=x` | Copy as-is |
| `$third-party` | `$third-party` | `$third-party` | Copy as-is |
| Scriptlet | `+js(name, args)` | `#%#//scriptlet("name", "args")` | Translate |
| HTML filtering | `##^selector` | `$$selector` | Different syntax |
| CSS injection | `#$#sel { css }` | `#$#sel { css }` | Mostly compatible |

## Repository Structure

```
my-adblock-filters/
├── README.md
├── .github/
│   └── workflows/
│       └── build.yml          # Auto-rebuilds flat files on push
├── scripts/
│   └── build.sh               # Concatenation + timestamp build script
├── ublock/
│   ├── all.txt                # Barrel file using !#include
│   ├── youtube.txt
│   ├── linkedin.txt
│   ├── microsoft365.txt
│   └── general.txt
└── adguard/
    ├── all.txt                # Barrel file using !#include
    ├── all-flat.txt           # Auto-generated concatenated version
    ├── youtube.txt
    ├── linkedin.txt
    ├── microsoft365.txt
    └── general.txt
```

## Maintenance

- **Keep uBO and AdGuard updated** to their latest versions for best filter compatibility
- **Purge filter caches periodically** — in uBO: Dashboard → Filter lists → Purge all caches → Update now
- **Use the element picker** (uBO's eyedropper tool or AdGuard's Assistant) to find new CSS selectors when sites update their layouts
- **Check the GitHub Action** status after pushing changes to ensure flat files were rebuilt correctly
- The build script updates `! Last modified:` timestamps automatically

## License

[MIT](LICENSE)
