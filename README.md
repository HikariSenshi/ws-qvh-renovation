# ws-qvh — renovation fork

> Fork of [NetrisTV/ws-qvh](https://github.com/NetrisTV/ws-qvh) that makes the iOS
> screen-streaming bridge **build and run on current macOS / iOS (including iOS 26)**.

## Why this fork exists

`ws-qvh` is the WebSocket bridge ws-scrcpy uses to stream an iPhone's screen — it
embeds [danielpaulus/quicktime_video_hack](https://github.com/danielpaulus/quicktime_video_hack)
to capture H.264 over USB. On a modern Mac the upstream binary is **dead on
arrival**: it is pinned to a 2020 dependency snapshot whose `gousb v2.1.0+incompatible`
panics at USB initialization against current libusb —

```
panic: libusb: unknown error [code -99]
```

— so ws-scrcpy's iOS streaming silently fails (the browser only shows
`[StreamReceiver] WS closed`), with no obvious reason why.

## What this fork changes

- **Builds against modern libusb** — re-points the `quicktime_video_hack` dependency
  at the upstream `externalizeGST` commit (`gousb v1.1.2`) and bumps to `go 1.19`,
  eliminating the `libusb -99` panic.
- **Removes dead code** (`getValues()` / `go-ios/usbmux`) that was unused and was the
  only thing pulling an incompatible `go-ios`, which blocked the rebuild.

**Result:** the rebuilt `ws-qvh` captures and streams H.264 from multiple iPhones on
**iOS 26.5 / Apple Silicon macOS** through ws-scrcpy.

The clean dependency fix is also contributed back upstream as
[NetrisTV/ws-qvh#20](https://github.com/NetrisTV/ws-qvh/pull/20).

## Build & use

```bash
brew install go libusb                 # prerequisites (also: Xcode command-line tools)
git clone https://github.com/HikariSenshi/ws-qvh-renovation.git
cd ws-qvh-renovation
CGO_ENABLED=1 go build -o ws-qvh .
cp ws-qvh ~/bin/                       # ws-scrcpy spawns `ws-qvh` by name from PATH
```

Then run ws-scrcpy (built with `INCLUDE_APPL`) and open an iOS device stream. The
original upstream setup steps below describe the full flow.

---

*Original upstream README follows.*

# ws qvh

Web Socket server for streaming the screen of iOS devices.

## How it works?

1. [danielpaulus/quicktime_video_hack](https://github.com/danielpaulus/quicktime_video_hack) - video streaming
2. [appium/WebDriverAgent](https://github.com/appium/WebDriverAgent) - device control
3. [NetrisTV/ws-scrcpy](https://github.com/NetrisTV/ws-scrcpy) - user interface
4. [NetrisTV/ws-qvh](https://github.com/NetrisTV/ws-qvh) - forwards the video stream over Web Socket

## Steps to set up

1. Get macOS (streaming only should also work on GNU/Linux)
2. Connect a device, accept "Trust This Computer".
3. Verify that you can record your device screen with QuickTime
4. Install [danielpaulus/quicktime_video_hack](https://github.com/danielpaulus/quicktime_video_hack) and verify that you can record your device screen with it
5. Build sources: `go build`. This command will produce `ws-qvh` binary.
6. Make sure your `ws-qvh` binary is available via the `PATH` environment variable.
7. Setup and run `ws-scrcpy`. Follow the instructions [here](https://github.com/NetrisTV/ws-scrcpy#ws-qvh).
8. Open link provided by `ws-scrcpy` in your browser.
   
# Notes

* Only video stream is transmitted (no audio).
* `WebDriverAgent` can be started only after the start of video transmission (i.e. quicktime interface activation).
* `WebDriverAgent` can take a long time to start.
* Control capabilities are very limited (compared to scrcpy/ws-scrcpy): 
   * single tap
   * `home` button click
   * swipe (this command will be sent only after the gesture is complete)
* No way to customize stream parameters (bitrate, fps, video size, etc.)
