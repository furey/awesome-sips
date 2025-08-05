# awesome-sips [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

A curated list of awesome resources, scripts, and examples for macOS `sips`, a macOS cli tool for scriptable image processing without external dependencies.

## Contents

- [Resources](#resources)
  - [Official Documentation](#-official-documentation)
  - [Guides & Tutorials](#-guides--tutorials)
- [Examples](#examples)
  - [Basic Operations](#-basic-operations)
  - [Batch Processing](#-batch-processing)
  - [Format Conversion](#-format-conversion)
  - [Image Manipulation](#-image-manipulation)
- [Scripts & Tools](#scripts--tools)
  - [Shell Scripts](#️-shell-scripts)
  - [Automation Tools](#-automation-tools)
- [Recipes](#recipes)
  - [Web Optimization](#-web-optimization)
  - [Photography Workflows](#-photography-workflows)
  - [Icon Generation](#-icon-generation)
- [Tips & Tricks](#tips--tricks)
- [Related Projects](#related-projects)
- [Contributing](#contributing)

## Resources

### 📚 Official Documentation

- `man sips` - The official man page (run in Terminal)

### 📖 Guides & Tutorials

- [`CHEATSHEET.md`](CHEATSHEET.md) - Comprehensive guide with visual examples
- [macOS image manipulation with sips](https://blog.smittytone.net/2019/10/24/macos-image-manipulation-with-sips/) - In-depth tutorial
- [Using sips for batch image processing](https://robservatory.com/use-sips-to-quickly-easily-and-freely-convert-image-files/) - The Robservatory guide
- [sips image processing](http://xahlee.info/img/sips_image_processing.html) - Xah Lee's tutorial

## Examples

### 🎯 Basic Operations

```bash
# Get image info
sips -g all image.jpg

# Quick resize
sips -Z 800 image.jpg --out resized.jpg

# Convert format
sips -s format png image.jpg --out image.png

# Rotate 90°
sips -r 90 image.jpg --out rotated.jpg
```

### 🔄 Batch Processing

```bash
# Convert all PNGs to JPEG
for f in *.png; do sips -s format jpeg "$f" --out "${f%.png}.jpg"; done

# Resize all images to max 1024px
sips -Z 1024 *.jpg --out resized/

# Create thumbnails
mkdir thumbs && sips -Z 150 *.jpg --out thumbs/
```

### 🔀 Format Conversion

```bash
# HEIC to JPEG (iPhone photos)
sips -s format jpeg photo.heic --out photo.jpg

# RAW to JPEG
sips -s format jpeg -Z 2000 photo.cr2 --out photo.jpg

# PDF from images
sips -s format pdf *.png --out combined.pdf
```

### ✨ Image Manipulation

```bash
# Flip horizontally (mirror)
sips -f horizontal image.jpg --out mirrored.jpg

# Crop to square from center
sips -c 500 500 image.jpg --out square.jpg

# Add padding
sips -p 600 600 --padColor FFFFFF image.jpg --out padded.jpg
```

## Scripts & Tools

### 🛠️ Shell Scripts

#### Web Image Optimizer

```bash
#!/bin/bash
# optimize_for_web.sh - Optimize images for web use
for img in "$@"; do
    sips -Z 1920 -s format jpeg -s formatOptions 85 \
         --optimizeColorForSharing "$img" \
         --out "web_${img%.*}.jpg"
done
```

#### Batch Watermark Adder

```bash
#!/bin/bash
# add_copyright.sh - Add copyright to images
year=$(date +%Y)
for img in *.jpg; do
    sips -s copyright "© $year Your Name" \
         -s artist "Your Name" "$img" \
         --out "watermarked/$img"
done
```

#### Smart Resizer

```bash
#!/bin/bash
# smart_resize.sh - Only resize images larger than target
target_width=1200
for img in *.jpg; do
    width=$(sips -g pixelWidth "$img" | awk '/pixelWidth/{print $2}')
    if [ "$width" -gt "$target_width" ]; then
        sips -Z "$target_width" "$img" --out "resized/$img"
    fi
done
```

### 🤖 Automation Tools

- **Folder Actions** - Automatically process images dropped in folders
- **Automator Workflows** - GUI-based sips automation
- **Hazel Rules** - Smart file processing with sips
- **Keyboard Maestro Macros** - Hotkey-triggered image processing

## Recipes

### 🌐 Web Optimization

#### Complete Web Workflow

```bash
#!/bin/bash
# Generate responsive image set
for size in 320 640 1024 1920; do
    sips -Z $size original.jpg --out "image-${size}w.jpg"
done

# Generate WebP alternatives (if supported)
# Note: sips can read but not write WebP
```

#### Social Media Presets

```bash
# Instagram Square (1080x1080)
sips -c 1080 1080 -p 1080 1080 --padColor FFFFFF image.jpg --out instagram.jpg

# Twitter Header (1500x500)
sips -z 500 1500 image.jpg --out twitter-header.jpg

# Facebook Cover (851x315)
sips -z 315 851 image.jpg --out fb-cover.jpg
```

### 📸 Photography Workflows

#### Contact Sheet Generator

```bash
#!/bin/bash
# Create uniform thumbnails for contact sheet
mkdir -p tiles
counter=0
for img in *.jpg; do
    sips -Z 200 -c 200 200 "$img" --out "tiles/$(printf "%04d" $counter).jpg"
    ((counter++))
done
# Use external tool to combine tiles into sheet
```

#### EXIF Metadata Preservation

```bash
# Process while preserving metadata
sips -Z 2000 -s format jpeg -s formatOptions 95 \
     -s copyright "© $(date +%Y) Your Name" \
     raw_file.cr2 --out processed.jpg
```

### 🎨 Icon Generation

#### macOS App Icon Set

```bash
#!/bin/bash
# Generate all required icon sizes
base="icon.png"
for size in 16 32 64 128 256 512 1024; do
    sips -z $size $size "$base" --out "icon_${size}x${size}.png"
done

# Create .icns (requires additional tools)
```

#### Favicon Set

```bash
# Generate favicon sizes
for size in 16 32 48; do
    sips -z $size $size logo.png --out "favicon-${size}.png"
done
```

## Tips & Tricks

### 💡 Performance

- Use `-Z` (capital) to maintain aspect ratio vs `-z` for exact dimensions
- Process images in parallel: `find . -name "*.jpg" | xargs -P 4 -I {} sips -Z 800 {} --out resized/{}`
- Pre-filter files to avoid unnecessary processing

### 🎯 Quality Settings

- JPEG: Use 85-95 for photos, 60-80 for web
- `--optimizeColorForSharing` for cross-device compatibility
- Preserve metadata with careful use of `-s` flags

### ⚡ Common Gotchas

- **No Undo** - sips overwrites immediately, always use `--out`
- **Path Spaces** - Always quote file paths: `"$file"`
- **WebP Limitation** - Can read but not write WebP format
- **Profile Paths** - Use full paths for ICC profiles

### 🔧 Pro Tips

```bash
# Check if image needs processing
[ $(sips -g pixelWidth "$img" | awk '/pixelWidth/{print $2}') -gt 1920 ] && sips -Z 1920 "$img"

# Extract embedded color profile
sips -x profile profile.icc image.jpg

# Remove color management (smaller files)
sips --deleteColorManagementProperties image.jpg --out stripped.jpg

# Quick image info
sips -g pixelWidth -g pixelHeight -g format *.jpg
```

## Related Projects

- [ImageMagick](https://imagemagick.org/) - Cross-platform alternative with more features
- [FFmpeg](https://ffmpeg.org/) - For video frame extraction to process with sips
- [exiftool](https://exiftool.org/) - Advanced metadata manipulation
- [pngquant](https://pngquant.org/) - PNG compression to pair with sips

## Contributing

Contributions welcome! Please:

1. Add useful scripts, examples, or resources
2. Ensure all code examples are tested on macOS
3. Follow the existing format
4. Include practical, real-world use cases

---

## License

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](https://creativecommons.org/publicdomain/zero/1.0/)

To the extent possible under law, the contributors have waived all copyright and related or neighboring rights to this work.
