# Visual Process Automation — A Look Back at Screen-Scraping RPA

> A personal project revisiting the computer-vision + coordinate-driven automation techniques used to integrate with systems that have no API — the same approach used to read mainframe "green screens" back in the early 2000s.

[![Python](https://img.shields.io/badge/Python-3.7+-blue.svg)](https://www.python.org/)
[![OpenCV](https://img.shields.io/badge/OpenCV-4.x-green.svg)](https://opencv.org/)
[![Selenium](https://img.shields.io/badge/Selenium-WebDriver-orange.svg)](https://www.selenium.dev/)

---

## Background

Back in 2004, I built screen-scrapers that read mainframe terminal screens and fed the extracted data into Java and C++ backends. There was no API — the mainframe only exposed a fixed-position character/UI screen — so the only way in was to read the screen itself, locate known fields by their position or visual pattern, and translate that into structured data the backend could consume.

This project revisits that same idea using modern tools: instead of parsing a terminal buffer, it uses **OpenCV template matching** to find UI elements on screen, and **Selenium + PyAutoGUI** to drive the interaction, in the same spirit as the old mainframe scrapers — find the element visually, act on its position, repeat.

## Why a browser game?

I needed a live target that:
- Had a UI that updates and shifts over time (icons, button states, layouts) — a good stress test for template matching robustness
- Ran continuously, so I could see how well the approach held up over a long unattended period, not just a quick demo

A browser-based game (Forge of Empires) was a convenient, constantly-changing UI to test against. It ran effectively unattended for **about two years**, clicking through the same categories of actions a screen-scraper would: locate an element by its visual template, resolve its on-screen coordinates, act on it, log the result, repeat.

This was purely a personal technical exercise in reviving/adapting an older automation technique — not intended as a tool for others to run, and not something I'd recommend pointing at a live third-party service; most platforms' terms of service prohibit this kind of automation, which is worth keeping in mind if you adapt this for your own experiments.

## What's actually in this repo

| File | Purpose |
|---|---|
| `main.py` / `main2.py` | Entry points — two iterations from different points in development |
| `bot.py` / `bot2.py` | Core automation loop — locate elements, decide what to click, act |
| `browser.py` / `browser2.py` | Selenium wrapper — launches Chrome, logs in, captures screenshots |
| `vision.py` | OpenCV template matching — finds "needle" images (UI elements) within the captured screenshot |
| `httprequest.py` | Early experiment calling the game's endpoints directly, as an alternative to pure visual automation |

**Note on the duplicate files:** `bot2.py`, `browser2.py`, and `main2.py` are earlier/parallel versions kept around during iteration — this predates my use of proper source control on personal projects, so instead of branches or commit history, I just copied and renamed files. They're left as-is to show the progression, not as a recommended pattern.

**Note on hardcoded values:** Window resolution, chromedriver path, and local profile directories are hardcoded rather than pulled from a config file. That wasn't the point of the exercise — the interesting part was the vision/matching approach — so I didn't bother externalizing it.

## How it works

1. **Screenshot capture** — Selenium grabs the current browser frame as an image.
2. **Template matching** — OpenCV (`cv.matchTemplate`) searches the screenshot for known "needle" images (buttons, icons, indicators) above a confidence threshold.
3. **Coordinate resolution** — matched positions are translated into actual screen coordinates, with per-element offset logic (since a button's clickable point isn't always its top-left match position).
4. **Action** — PyAutoGUI moves the mouse and clicks, mimicking a human interacting with the same UI.
5. **Logging** — every match, click, and state transition is logged for later review of how reliably the matching held up over time.

This is the direct modern equivalent of the mainframe scraper: no API, no DOM hooks — just "find it visually, act on its position."

## What this technique is good for

- Legacy terminal/mainframe applications with no API (the original use case)
- Third-party desktop or web software with no automation hooks
- Canvas-rendered UIs where DOM selectors don't apply
- Quick proof-of-concept automation before investing in a "real" API integration

## Limitations (learned the hard way over ~2 years of runtime)

1. **Resolution-dependent** — needle images must be captured at the same resolution the target is running at; any mismatch breaks matching.
2. **Brittle to UI changes** — any visual redesign, even a minor icon refresh, silently breaks matching until the template is recaptured.
3. **No semantic understanding** — the bot has no idea *what* a button does, only that pixels in a region resemble a stored template. All the "meaning" is hardcoded into the click-target-offset logic per element name.
4. **Confidence tuning is fiddly** — too strict and legitimate matches get missed; too loose and false positives fire. This needed per-element tuning (visible in the varying confidence thresholds throughout `bot.py`).
5. **No error recovery beyond retry** — if the UI is in an unexpected state, the bot has no way to reason about it; it just times out or clicks the wrong thing.

These are exactly the same limitations mainframe screen-scrapers had in the 2000s — position/pattern-based automation is inherently fragile to changes in the thing it's watching. Nothing about moving to a modern browser target changed that fundamental tradeoff, which was really the point of doing this exercise: fun to see it hold up in principle, but a good reminder of why API access is always preferable when it's available.

---

## Requirements

```
opencv-python>=4.5.0
numpy>=1.19.0
selenium>=4.0.0
pillow>=8.0.0
pyautogui>=0.9.50
```

## License

MIT — personal/educational use. No warranty, no liability, not intended for production use against third-party services.
