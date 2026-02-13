# Ghostty Themed Icons

This repo generates themed Ghostty app icons for macOS using color palettes from terminal themes.

## Quick Reference

When the user says **"create a new icon using \<theme\> palette"**, follow the full pipeline below.

## Project Structure

```
templates/
  icon-composer.icon/       # .icon template with Icon Composer group settings
  icon.svg                  # Flat single-file SVG template (all layers composited)
icons/
  {theme}/
    {theme}.icon/           # Icon Composer package (3 SVG layers + icon.json)
    {theme}.png             # Full-size 1024x1024 PNG rendered via ictool
    {theme}.icns            # Final padded macOS icon (from /create-icns skill)
```

## Theme Color Mapping

Each theme needs 6 hex colors:

| Role | Description | Maps to |
|------|-------------|---------|
| `bg` | Background color | Background layer fill, icon.json top-level fill |
| `fg` | Foreground color | Ghost layer fill, dark mode fill specialization |
| `accent1` | First accent (typically red/pink) | Color bar — left segment (rounded) |
| `accent2` | Second accent (typically blue/cyan) | Color bar — second segment |
| `accent3` | Third accent (typically green) | Color bar — third segment |
| `accent4` | Fourth accent (typically yellow/gold) | Color bar — right segment (rounded) |

## Full Pipeline

### Step 1: Get Theme Colors

Resolve the 6 hex colors needed (bg, fg, accent1–4):

1. **Check the palette table** at the bottom of this file — if the theme is already listed, use those colors and skip to Step 2.
2. **Look up the theme** — search the web for the theme's official color palette (GitHub repos, official docs, or theme gallery sites like `github.com/mbadolato/iTerm2-Color-Schemes`). Identify:
   - `bg` → the theme's main background color
   - `fg` → the theme's main foreground/text color
   - The full set of ANSI/accent colors available in the palette
3. **Propose accent colors for approval** — present the user with a suggested set of 4 accent colors for the color bar, along with your reasoning. Consider the theme's **iconic/signature colors** — e.g., Dracula is known for purple, Nord for its frost blues, Rosé Pine for its rose and gold. The bar reads left to right so arrange for good visual balance:
   - `accent1` (left, rounded) → typically a warm color (red, pink, magenta)
   - `accent2` → a cool/mid color (blue, cyan, purple)
   - `accent3` → a natural color (green, teal)
   - `accent4` (right, rounded) → a warm accent (yellow, gold, orange)

   Show the hex values with color names so the user can evaluate. Offer alternatives from the palette if there are multiple good candidates (e.g., "Dracula has both purple `#bd93f9` and cyan `#8be9fd` — I'd suggest cyan for accent2 since it's a signature Dracula color, but purple is also available").
4. **Wait for user approval** before proceeding. The user may swap colors or reorder them.
5. **Ask the user** for colors directly only if the theme can't be found online.

After approval, add the theme to the **Existing Theme Palettes** table at the bottom of this file.

### Step 2: Generate SVG Layers

Create 3 SVGs in `icons/{slug}/{slug}.icon/Assets/`:

**`{slug}-1-background.svg`** — solid background:
```svg
<svg xmlns="http://www.w3.org/2000/svg" width="1024" height="1024" viewBox="0 0 1024 1024" fill="none"><rect width="1024" height="1024" fill="{bg}" /></svg>
```

**`{slug}-2-color-bar.svg`** — 4-color pill bar:
```svg
<svg xmlns="http://www.w3.org/2000/svg" width="1024" height="1024" viewBox="0 0 1024 1024" fill="none"><path d="M181 793.5C181 762.848 205.848 738 236.5 738H348V849H236.5C205.848 849 181 824.152 181 793.5V793.5Z" fill="{accent1}" /><rect x="348" y="738" width="167" height="111" fill="{accent2}" /><rect x="515" y="738" width="167" height="111" fill="{accent3}" /><path d="M682 738H793.5C824.152 738 849 762.848 849 793.5V793.5C849 824.152 824.152 849 793.5 849H682V738Z" fill="{accent4}" /></svg>
```

**`{slug}-3-ghost.svg`** — ghost terminal logo:
```svg
<svg xmlns="http://www.w3.org/2000/svg" width="1024" height="1024" viewBox="0 0 1024 1024" fill="none"><path d="M333.035 163.598C416.84 163.598 484.793 231.552 484.793 315.355V491.63C484.793 518.608 463.984 541.439 437.039 542.924C423.954 543.632 411.882 539.382 402.54 531.861C391.344 522.858 375.595 523.262 364.297 532.165C355.697 538.943 344.839 542.991 333.002 542.991C321.164 542.991 310.339 538.943 301.739 532.165C290.307 523.161 274.558 523.161 263.126 532.165C254.594 538.877 243.87 542.924 232.201 542.991C204.177 543.193 181.278 519.586 181.278 491.562V315.355C181.278 231.552 249.232 163.598 333.035 163.598ZM246.536 269.377C239.184 265.128 229.741 267.657 225.492 275.009C221.243 282.361 223.772 291.803 231.124 296.053L264.578 315.377L231.124 334.701C223.773 338.949 221.243 348.358 225.492 355.743C229.741 363.095 239.151 365.624 246.536 361.375L303.09 328.73C313.342 322.795 313.342 307.992 303.09 302.055V302.021L246.536 269.377ZM356.846 299.935C348.348 299.935 341.435 306.814 341.435 315.346C341.435 323.878 348.314 330.758 356.846 330.758H431.14C439.638 330.758 446.551 323.878 446.551 315.346C446.551 306.814 439.672 299.935 431.14 299.935H356.846Z" fill="{fg}" /></svg>
```

### Step 3: Generate the .icon Package

Create `icons/{slug}/{slug}.icon/icon.json`. Convert hex colors to display-p3 format by dividing each RGB component by 255 (5 decimal places): `display-p3:R,G,B,1.00000`.

Use the group settings from `templates/icon-composer.icon/icon.json` as the template. The structure has 3 groups:

1. **Group 1 (front) — Ghost**: fill-specializations for dark mode with fg color, individual lighting in dark, no shadow in dark, specular enabled in dark, translucency disabled in dark
2. **Group 2 (middle) — Color bar**: neutral shadow, specular disabled (both appearances), translucency disabled (both appearances). Note: specializations need entries for BOTH default (no `appearance` key) and dark (`"appearance": "dark"`)
3. **Group 3 (back) — Background**: neutral shadow, translucency enabled (0.5)

Update these fields per theme:
- Top-level `fill.automatic-gradient` → bg color in display-p3
- Ghost layer `fill-specializations[dark].automatic-gradient` → fg color in display-p3
- All `image-name` and `name` fields → `{slug}-{n}-{layer}.svg`

### Step 4: Export PNG via ictool

```bash
"/Applications/Xcode.app/Contents/Applications/Icon Composer.app/Contents/Executables/ictool" \
  icons/{slug}/{slug}.icon \
  --export-image \
  --output-file icons/{slug}/{slug}.png \
  --platform macOS \
  --rendition Default \
  --width 1024 \
  --height 1024 \
  --scale 1
```

### Step 5: Create .icns with Padding

Add Tahoe-style padding (scale content to ~835x835, centered on 1024x1024 transparent canvas) and generate the `.icns`:

```bash
# Add padding
magick icons/{slug}/{slug}.png -resize 835x835 -gravity center -background none -extent 1024x1024 /tmp/{slug}-padded.png

# Generate all required icon sizes
mkdir -p /tmp/icon.iconset
sips -z 16 16     /tmp/{slug}-padded.png --out /tmp/icon.iconset/icon_16x16.png
sips -z 32 32     /tmp/{slug}-padded.png --out /tmp/icon.iconset/icon_16x16@2x.png
sips -z 32 32     /tmp/{slug}-padded.png --out /tmp/icon.iconset/icon_32x32.png
sips -z 64 64     /tmp/{slug}-padded.png --out /tmp/icon.iconset/icon_32x32@2x.png
sips -z 128 128   /tmp/{slug}-padded.png --out /tmp/icon.iconset/icon_128x128.png
sips -z 256 256   /tmp/{slug}-padded.png --out /tmp/icon.iconset/icon_128x128@2x.png
sips -z 256 256   /tmp/{slug}-padded.png --out /tmp/icon.iconset/icon_256x256.png
sips -z 512 512   /tmp/{slug}-padded.png --out /tmp/icon.iconset/icon_256x256@2x.png
sips -z 512 512   /tmp/{slug}-padded.png --out /tmp/icon.iconset/icon_512x512.png
cp                /tmp/{slug}-padded.png /tmp/icon.iconset/icon_512x512@2x.png

# Convert to .icns
iconutil -c icns /tmp/icon.iconset -o icons/{slug}/{slug}.icns

# Clean up
rm -rf /tmp/icon.iconset /tmp/{slug}-padded.png
```

Show the final .icns to the user for verification.

## Existing Theme Palettes

| Theme | bg | fg | accent1 | accent2 | accent3 | accent4 |
|-------|----|----|---------|---------|---------|---------|
| Tokyo Night | #1a1b26 | #c0caf5 | #ff9e64 | #9ece6a | #bb9af7 | #7aa2f7 |
| Tokyo Night Storm | #24283b | #c0caf5 | #ff9e64 | #9ece6a | #bb9af7 | #7aa2f7 |
| Tokyo Night Moon | #222436 | #c8d3f5 | #ff966c | #c3e88d | #c099ff | #82aaff |
| Tokyo Night Day | #e1e2e7 | #3760bf | #b15c00 | #587539 | #9854f1 | #2e7de9 |
| Catppuccin Mocha | #1e1e2e | #cdd6f4 | #fab387 | #a6e3a1 | #cba6f7 | #89b4fa |
| Catppuccin Macchiato | #24273a | #cad3f5 | #f5a97f | #a6da95 | #c6a0f6 | #8aadf4 |
| Catppuccin Frappé | #303446 | #c6d0f5 | #ef9f76 | #a6d189 | #ca9ee6 | #8caaee |
| Catppuccin Latte | #eff1f5 | #4c4f69 | #fe640b | #40a02b | #8839ef | #1e66f5 |
| Dracula | #282a36 | #f8f8f2 | #ff79c6 | #bd93f9 | #50fa7b | #f1fa8c |
| Nord | #2e3440 | #eceff4 | #d08770 | #81a1c1 | #88c0d0 | #a3be8c |
| Rosé Pine | #191724 | #e0def4 | #eb6f92 | #9ccfd8 | #31748f | #f6c177 |
| Solarized Dark | #002b36 | #839496 | #dc322f | #268bd2 | #859900 | #b58900 |
| Solarized Light | #fdf6e3 | #657b83 | #dc322f | #268bd2 | #859900 | #b58900 |
| Everforest Dark Hard | #272e33 | #d3c6aa | #e69875 | #a7c080 | #83c092 | #dbbc7f |
| Everforest Dark Medium | #2d353b | #d3c6aa | #e69875 | #a7c080 | #83c092 | #dbbc7f |
| Everforest Dark Soft | #333c43 | #d3c6aa | #e69875 | #a7c080 | #83c092 | #dbbc7f |
| Everforest Light Hard | #fffbef | #5c6a72 | #f57d26 | #8da101 | #35a77c | #dfa000 |
| Everforest Light Medium | #fdf6e3 | #5c6a72 | #f57d26 | #8da101 | #35a77c | #dfa000 |
| Everforest Light Soft | #f3ead3 | #5c6a72 | #f57d26 | #8da101 | #35a77c | #dfa000 |
| Kanagawa Wave | #1f1f28 | #dcd7ba | #e46876 | #7e9cd8 | #98bb6c | #e6c384 |
| Kanagawa Dragon | #181616 | #c5c9c5 | #c4746e | #8ba4b0 | #87a987 | #c4b28a |
| Kanagawa Lotus | #f2ecbc | #545464 | #c84053 | #6693bf | #6f894e | #de9800 |
| Poimandres | #1b1e28 | #e4f0fb | #5DE4C7 | #ADD7FF | #D0679D | #FFFAC2 |
| Poimandres Storm | #303340 | #e4f0fb | #5DE4C7 | #ADD7FF | #D0679D | #FFFAC2 |
