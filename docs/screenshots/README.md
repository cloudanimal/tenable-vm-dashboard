# Screenshots

`dashboard.png` is the full-page hero used in the top-level README. It is **generated reproducibly** from the bundled synthetic sample data (no real environment), so it can be regenerated whenever the UI changes:

```bash
# 1. serve the repo
python3 -m http.server 8755

# 2. headless render of the populated dashboard, trimmed to content height (2× for retina)
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --hide-scrollbars --force-device-scale-factor=2 \
  --window-size=1500,3600 --virtual-time-budget=20000 \
  --screenshot=docs/screenshots/dashboard.png \
  "http://localhost:8755/?autosample=1"
```

`?autosample=1` loads the bundled sample on page load (the same convenience flag works for demo links). Adjust `--window-size` height if the page grows. Dark theme is the default.
