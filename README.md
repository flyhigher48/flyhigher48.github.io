# Notes from a Working Engineer — GitHub Pages (v5)

This version includes:
- Site title updated
- Default skin set to **air** (see `_config.yml` to try: aqua, contrast, dark, dirt, focus, mint, neon, plum, sunrise)
- Default social image at `/assets/images/social/default.png` (PNG + SVG)
- SEO defaults: every post uses the default social image unless you set `image:` in the post
- Custom social share buttons with current X/Facebook/LinkedIn icons
- Topic landing pages + navigation

### Change the accent color (skin) on the fly
Edit `_config.yml` and set:
```
minimal_mistakes_skin: "aqua"     # try "contrast", "mint", "sunrise", etc.
```
Commit to `main` and refresh. No other changes required.
