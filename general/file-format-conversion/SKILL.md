---
name: file-format-conversion
description: 
category: general
tags: [file-format-conversion]
---

## When to Use
Batch convert files: ffmpeg for video/audio, imagemagick for images, pandoc for docs.

## FFmpeg
```bash
# Video to GIF
ffmpeg -i input.mp4 -vf "fps=10,scale=480:-1" output.gif

# Extract audio
ffmpeg -i input.mp4 -vn -acodec libmp3lame output.mp3

# Resize video
ffmpeg -i input.mp4 -vf scale=1280:720 output.mp4

# Batch convert
for f in *.mp4; do
    ffmpeg -i "$f" -c:v libx264 -crf 23 "${f%.mp4}.mkv"
done
```

## ImageMagick
```bash
# Resize
convert input.jpg -resize 800x600 output.jpg

# Batch watermark
mogrify -path output/ -gravity SouthEast -draw "text 10,10 '© 2024'" *.jpg

# Optimize PNG
pngquant --quality=65-80 input.png -o output.png
```

## Pandoc
```bash
# Markdown to HTML
pandoc input.md -o output.html

# Word to Markdown
pandoc input.docx -o output.md

# PDF generation
pandoc input.md -o output.pdf --pdf-engine=xelatex
```

## Pitfalls
- **Quality loss**: Use -q or -crf flags carefully
- **Format support**: Check codec availability
- **Batch safety**: Test on one file first
- **File paths**: Handle spaces in filenames

## Verification
- Compare output quality with source
- Check file sizes are reasonable
- Test with various input formats