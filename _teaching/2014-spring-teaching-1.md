---
title: "A short macOS terminal script to automatically open a texthooker and Manga OCR."
collection: teaching
date: 2024-06-02
excerpt: "I made a short console script to automatically open a text hooker and Manga OCR. With just a few lines of code, it becomes easier to jump into reading manga at any time."
---

I made a short console script to automatically open a text hooker and Manga OCR. With just a few lines of code, it becomes easier to jump into reading manga at any time.

The code is a simple script that opens Google Chrome, maximizes the window to full screen, opens an Anacreon text hooker, and finally runs the Python Manga OCR script on a pre-specified system path to read the screenshots.

```
osascript -e 'tell application "Google Chrome"
activate
  make new window
  tell application "System Events"
    keystroke "f" using {control down, command down}
  end tell
end tell'
open https://anacreondjt.gitlab.io/texthooker.html

python -m manga_ocr "path" # specify path as needed
```
