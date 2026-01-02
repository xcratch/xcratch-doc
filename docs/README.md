# What is Xcratch

[Xcratch](https://xcratch.github.io/) is a mod application of [Scratch3 by MIT](https://scratch.mit.edu/) to play with user-made extensions easily.

## Features

### On-demand installation and auto-loading

You can install and use user-made extensions from the web. When you open a project that uses extended blocks, Xcratch automatically installs the required extensions. Extensions are organized with tags so you can filter and search for what you need.

### Preloaded extensions

The Xcratch editor ships with several handy extensions preloaded, so you can use them even when you are offline.

### Web Bluetooth API support

You don't need to run [Scratch Link](https://scratch.mit.edu/microbit) to play with micro:bit or other BLE devices when your browser supports the Web Bluetooth API (Chrome, Edge, Chromebook browser, [Scrub](https://apps.apple.com/jp/app/scrub-web-browser/id1569777095) on iPad, and similar).

### Available offline

The Xcratch editor is a PWA (Progressive Web Apps). Once you load it, you can keep using it without further Internet access. Some extensions may still require connectivity. You can also install the editor on your local machine if needed.

## Local Backpack

Xcratch includes a backpack feature so you can store sprites, costumes, sounds, and code blocks for easy reuse across projects. The backpack uses your browser's local storage, so the contents remain available even when you are offline.

### Open project from URL

Xcratch can open projects that are published on the Internet directly. There are two ways to do it:

- Open the editor with `#`: https://xcratch.github.io/editor/#<project-URL>
- Add a `project` query parameter: https://xcratch.github.io/editor/?project=<project-URL>

### UI improvements from Scratch Foundation version

- Numbers entered with double-byte characters can be used as numerical values.
- Japanese arithmetic operators use clearer symbols (×, ÷, ＋, −).
- New line (\n) and tab (\t) characters can be used for "say", "think", and "=" decisions.
- Code clean-up improves horizontal alignment and animations.
- Comment position bugs that caused comments to move have been fixed.
- You can see the hidden area on the right edge of the block palette.
- Backpack can be used on your own server.
- When embedded in HTML, the editor can be opened and full-screen mode is supported.
