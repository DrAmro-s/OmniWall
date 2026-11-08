# OmniWall Pro — Master Architecture & Feature Specification

> **OmniWall Pro: Master Edition** is an ultra-high-performance, GPU-accelerated desktop media dashboard, infinite canvas workstation, and multi-layer reactive live wallpaper engine running on Electron and Chromium. Designed as a zero-dependency, self-contained architecture, it achieves zero-latency local playback, gapless audio-video decoding, direct disk streaming, and deep hardware-composited styling.

---

## Table of Contents
1. [Core Architecture & Design Philosophy](#1-core-architecture--design-philosophy)
2. [Workspace Canvas & Layout Management](#2-workspace-canvas--layout-management)
3. [Drag & Drop Intelligent Ingestion Engine](#3-drag--drop-intelligent-ingestion-engine)
4. [Local Video Player Tiles](#4-local-video-player-tiles)
5. [Image Slideshow & Gallery Tiles](#5-image-slideshow--gallery-tiles)
6. [Web Video & Streaming Tiles (YouTube / Vimeo / Piped)](#6-web-video--streaming-tiles)
7. [Universal Audio Player Suite](#7-universal-audio-player-suite)
8. [Multi-Layer Reactive Live Wallpaper Engine](#8-multi-layer-reactive-live-wallpaper-engine)
9. [Looks & Visual Styling System](#9-looks--visual-styling-system)
10. [State Persistence & `.omniwall` Format](#10-state-persistence--omniwall-format)
11. [Master Keyboard & Mouse Shortcut Reference](#11-master-keyboard--mouse-shortcut-reference)
12. [Future Architectural Roadmap & Ideas](#12-future-architectural-roadmap--ideas)
13. [Getting Started & Installation](#13-getting-started--installation)

---

## 1. Core Architecture & Design Philosophy

### 1.1 The Single-File Zero-Dependency Engine
* **Self-Contained Distribution:** The entire user interface, rendering logic, Web Audio nodes, styling tokens, and interaction event loops reside in a single cohesive frontend runtime (`OmniWall.html`).
* **Hardware-Accelerated Compositing:** Every media tile operates on an isolated composite GPU layer using CSS `will-change: transform`, `translateZ(0)`, and `contain: layout paint`. This isolates UI redraws from video decoding cycles, guaranteeing smooth 60 FPS pan and zoom even with 20+ simultaneous 4K streams playing.
* **Direct Disk Streaming (Zero AppData Bloat):** Bypasses traditional IndexedDB caching and electron cache cloning. Video and image files stream directly from host NVMe/SATA storage paths via sanitized `file:///` protocols, eliminating disk duplication.

### 1.2 VisionOS Liquid Glass Interface System
* **Deep Specular Blur:** Windows and floating HUD bars use multi-stop linear gradients combined with `backdrop-filter: blur(28px) saturate(190%) contrast(104%)`.
* **Crisp Low-Blur Hairlines:** 1px top-lit specular highlights (`rgba(255, 255, 255, 0.28)`) simulate physical refraction along bevel edges, ensuring readable contrast against both pure black backgrounds and high-luminance video content.
* **Non-Blocking Translucency:** HUDs and tile headers auto-fade when inactive and never obscure user content during fullscreen presentations.

---

## 2. Workspace Canvas & Layout Management

### 2.1 Infinite Coordinate Plane
* **10,000 × 10,000px Virtual Canvas:** Freely place media across an expansive coordinate plane with sub-pixel positioning accuracy.
* **Dynamic Z-Index Stacking:** Clicking any tile promotes it and all currently co-selected tiles to the top of the stack (`++state.topZ`) while preserving relative layering among selected items.
* **Clean Presentation Mode (`H`):** Instantly suppresses all UI overlays, tile control footers, top HUD bars, resize rails, modal windows, and audio drawers for pure media display.

### 2.2 Alignment & Spatial Packing
* **Magnetic Smart Snapping (Hold `Ctrl` while dragging/resizing):**
  * Snaps tile edges and centers to neighboring tiles within a 15px threshold.
  * Snaps to canvas screen borders and viewport centers.
  * Axis Locking: Holding `Ctrl` while moving constrains motion to strictly horizontal or strictly vertical axes based on initial movement vector.
  * Aspect Ratio Locking: Holding `Ctrl` while dragging resize handles locks the media source aspect ratio.
* **Eliminate Space Engine:**
  * Analyzes all active or selected tiles and sorts them by canvas position.
  * Dynamically calculates overlaps and collapses horizontal and vertical gaps, packing cards side-by-side with zero pixel wastage.

### 2.3 Non-Destructive History (Undo / Redo)
* **35-Step Deep State Snapshots:** Tracks moves, resizes, playlist index changes, styling updates, and deletions.
* **Pre-Drag Commit Guarantee:** Snapshots are captured before cursor displacement begins; releasing a drag pushes the exact origin coordinate into the undo stack, making `Ctrl+Z` instant and reliable.

---

## 3. Drag & Drop Intelligent Ingestion Engine

OmniWall Pro features an intelligent ingestion pipeline capable of differentiating single files, multi-file selections, and recursive directory structures:

```
                      [ Drag & Drop Ingestion ]
                                  │
          ┌───────────────────────┴───────────────────────┐
          ▼                                               ▼
   [ Folder Ingestion ]                          [ Loose File Ingestion ]
          │                                               │
   Recursive Scan                                  File Type Check
          │                                               │
   ┌──────┴──────┐                                 ┌──────┴──────┐
   ▼             ▼                                 ▼             ▼
[Videos]     [Images]                           [Video]       [Image]
   │             │                                 │             │
1 Playlist    1 Slideshow                       Independent   Independent
Tile Spawned  Tile Spawned                      Tile Spawned  Tile Spawned
```

### 3.1 Folder Drop Behavior
* Dropping a directory triggers a recursive `webkitGetAsEntry` traversal.
* Media files are sorted numerically and alphabetically using natural collation (`naturalCollator`).
* **Smart Consolidation:** 
  * All video files within the folder bundle into a single **Video Playlist Tile**.
  * All image files within the folder bundle into a single **Image Slideshow Tile**.
* Prevents canvas flooding when dropping folders containing hundreds of sequential assets.

### 3.2 Loose File Selection Behavior
* Dropping multiple loose files outside a folder automatically spawns each media file as an **independent tile**.
* Tiles are arranged in an offset cascade across the viewport, ready for individual resizing and positioning.

### 3.3 Audio Ingestion
* Dropping files or folders onto the **Top HUD Music Icon**, the **Sliding Audio Sub-Bar**, or the **Audio Setup Modal** routes tracks straight into the **Universal Audio Player Queue**, keeping them off the visual canvas.

### 3.4 File Path Normalization & Auto-Healing
* Resolves duplicate directory paths (e.g., `C:/Media/Media/video.mp4` $\rightarrow$ `C:/Media/video.mp4`).
* Automatically formats Windows drive paths into standards-compliant URI components (`file:///C:/...`).

---

## 4. Local Video Player Tiles

```
┌─────────────────────────────────────────────────────────────┐
│ 🎬 12  Cinematic_Sequence_01.mp4                 [⚙][✕]    │ Tile HUD
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    Active Video Viewport                    │
│                 (Dual-Buffer Gapless Swap)                  │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│ [▶] ═══════════════════════════════════════ 01:24 / 04:30   │ Tile Footer
│ Vol: ━━━━━●━━━━  Opac: ━━━━━━━━━━●                          │
└─────────────────────────────────────────────────────────────┘
```

### 4.1 Dual-Buffer Seamless Engine (`vidA` & `vidB`)
* Every video tile contains two synchronized HTML5 video elements.
* While `vidA` plays, `vidB` preloads the next playlist entry in the background.
* Upon track completion, opacities and z-indexes swap in 0 ms, eliminating black flashes, decoder stutters, and loading gaps.

### 4.2 Frame-Accurate Paused Seeking & Scrubber Sync
* **Synchronized Scrubbing:** Moving through frames while paused via keyboard arrows immediately updates the timeline slider and time stamps without requiring playback to start.
* **Adaptive Fast Seeking:** Uses browser `fastSeek()` where supported, with sub-millisecond precision fallbacks.
* **Instant Frame-Zero Rewind (`Ctrl + 0`):** Jumps immediately to frame `00:00.000` on the active video tile.

### 4.3 Fit Modes & Proportional Unification
* **Stretch to Fill (`fill`):** Fills tile bounds completely without letterboxing.
* **Scale to Fit (`contain`):** Preserves native source aspect ratio inside tile borders with black framing.
* **Scale to Fill (`cover`):** Fills card dimensions while cropping excess frame edges.
* **Unify Aspect Ratio (`U`):** Automatically resizes the tile's outer dimensions to match the source video's native width and height ratio, eliminating letterboxing.

### 4.4 In-Card Control Bar
* **Seek Timeline:** Interactive red scrub line displaying current vs. total duration.
* **Volume Control:** Direct volume level slider with click-to-mute.
* **Card Opacity:** Controls the tile's master alpha channel (10% to 100%).

---

## 5. Image Slideshow & Gallery Tiles

### 5.1 High-Speed In-Memory Blob Streaming
* Reads image assets directly via `URL.createObjectURL()`, bypassing file-system access limitations and avoiding disk cloning.
* Auto-adapts dimensions on every slide transition to match native vertical or horizontal photo orientations.

### 5.2 Transition & Timing System
* **Slide Interval:** Adjustable between **0.05 seconds** (high-speed rapid sequence) and **120 seconds**.
* **Crossfade Transition Engine:** Smooth alpha-blending crossfade between incoming and outgoing images with durations adjustable from 0.05s to 1.5s.

### 5.3 Dynamic Framing & Motion Effects
1. **Zoom In:** Slow, cinematic Ken Burns push inward towards the center.
2. **Zoom Out:** Gradual pull back from high scale down to standard fit.
3. **Tile Grow:** The entire tile container expands along with the image.
4. **Static / None:** Standard presentation with no movement.

---

## 6. Web Video & Streaming Tiles

### 6.1 Multi-Source Embed Parser
* Automatically parses and sanitizes:
  * Standard YouTube links (`youtube.com/watch?v=...`)
  * YouTube Shorts (`youtube.com/shorts/...`)
  * Shortened URLs (`youtu.be/...`)
  * Vimeo streams (`vimeo.com/...`)
  * Direct web streams (`.mp4`, `.webm`, `.mkv`, `.mov`)

### 6.2 Bypass & Ad-Resistant Routing
* Prepending `!` or `bypass:` to a URL loads the stream through an alternative player interface (e.g., Piped embed), bypassing bot verifications and login walls for age-restricted videos.

### 6.3 Dual Interaction Modes
* **🎮 Interact Mode:** Full mouse and keyboard interaction inside the iframe (navigating YouTube controls, quality menus, comments, and scrubbing).
* **✋ Move Mode:** Drops a transparent drag shield across the iframe surface, allowing clicks anywhere on the tile to drag, move, snap, or select it on the canvas.

---

## 7. Universal Audio Player Suite

```
  Top HUD Bar: [💾 Layouts] [🎨 Looks] [🖼 Wallpaper] [🎵 Audio] [🌐 Web]
                                   │
┌──────────────────────────────────┴────────────────────────────────────┐
│  Omni Music — Artist - Track Name.mp3 (3s Stationary Marquee)         │
│  [⏮] [▶] [⏭]  02:15 ═══════════════════════ 04:45  🔊 [══●══] 150% [▼] │
└───────────────────────────────────────────────────────────────────────┘
```

### 7.1 VisionOS Sliding Sub-Bar
* Retracts smoothly beneath the top HUD pill.
* Features a 3-second stationary resting marquee that pauses before scrolling long track titles.

### 7.2 Perceptual 200% Loudness Boost Engine
* Integrated **Web Audio API** pipeline:
  $$\text{AudioSource} \longrightarrow \text{GainNode} \longrightarrow \text{AudioDestination}$$
* Allows volume boost up to **200%**, applying a non-linear perceptual power curve ($1.0 + (V - 1.0)^{1.2} \times 1.8$) for clear amplification without digital clipping.

### 7.3 Side-by-Side Queue & Peek Drawer
* **Queue List:** Displays up to 4 items in a scrollable view with album artwork, title, and duration.
* **Synchronized Video Peek:** Integrated video monitor displaying visuals for audio-video files alongside the audio stream.
* **YouTube Scaled Peek:** Scales YouTube video frames to 65% size to keep web player controls clean and compact.

### 7.4 Auto-Retracting Idle Guard
* Automatically retracts the drawer after **6 seconds of inactivity**.
* Pauses the timer whenever the user hovers over the drawer, scrolls, or interacts with the video peek monitor.

### 7.5 Queue Management
* Multi-criteria sorting: Name (A-Z / Z-A), Date Added (New-Old / Old-New), and Shuffle.
* Dedicated **Clear Playlist** button with warning styling to prevent accidental queue wipes.
* Loop modes: Toggle between **Repeat Single Track** and **Loop Entire Playlist**.

---

## 8. Multi-Layer Reactive Live Wallpaper Engine

```
┌─────────────────────────────────────────────────────────────────────┐
│                       Wall Display Canvas                           │
│  ┌───────────┐       ┌──────────────────────┐       ┌───────────┐  │
│  │  Mirror   │ <===  │     Center Slot      │  ===> │  Mirror   │  │
│  │   Wing    │       │     Active Media     │       │   Wing    │  │
│  └───────────┘       └──────────────────────┘       └───────────┘  │
│   (Stacked Blur)       (100% Native Aspect)          (Flipped Refl) │
└─────────────────────────────────────────────────────────────────────┘
```

### 8.1 Multi-Tab Layer Profiles
* Run multiple independent wallpaper layers simultaneously, each assigned a custom color tab.
* Layer individual tabs as background videos, ambient textures, or streaming web layers with independent opacities and visual filters.

### 8.2 60 FPS Hardware Canvas Mirror Wings
* Eliminates vertical video black bars (pillarboxing) on widescreen monitors.
* Dynamically clones the edge pixels of the playing video onto outer HTML5 canvases via `requestVideoFrameCallback`.
* **Reflect Inversion:** Horizontally mirrors wings for a seamless reflection look.
* **Configurable Wings Stack:**
  * Wing count: Up to 5 mirrored slots per side.
  * Mirror scale, offsets, blur, and brightness adjustments.

### 8.3 YouTube Ambient Stream Mode
* Embeds 24/7 live ambient streams (e.g., lo-fi hip-hop, rain sounds, space feeds).
* **Interactive Gear Mode (⚙):** Elevates the YouTube stream's z-index and unlocks mouse interactions so you can set stream quality to 1080p or 4K, then drops back to desktop level with settings preserved.

### 8.4 Framing, Visual Filters & Dynamic Pulse
* **Framing Controls:** Real-time Pan X/Y offsets ($\pm 50\%$) and Zoom Scale ($10\%$ to $300\%$).
* **Filters:** Real-time Brightness, Saturation, Dimming, and Blur sliders.
* **Dynamic Pulse:** Automatically pulses brightness between custom minimum and maximum levels using a smooth sine wave cycle (0.3s to 60s).

---

## 9. Looks & Visual Styling System

### 9.1 Target vs. All Scope Controller
The Looks Panel provides a global scope switch:
* **Scope: ALL (Default):** Changes apply across **every tile** on the canvas, overriding individual looks for a unified aesthetic.
* **Scope: TARGET:** Changes apply **only to currently selected tiles**, allowing for customized accent cards.

### 9.2 Styling Controls
* **Corner Curvature:** Adjustable from 0px (sharp square) to 40px (curved pill) with quick presets for 0px, 8px, and 18px.
* **Specular Stroke Outline:** Custom color picker, width slider (1px to 20px), and opacity control.
* **Alpha Edge Feathering:**
  * Shapes: **Oval / Elliptical** or **Rectangular**.
  * Controls: Center Depth (inner solid area), Depth of Field (softness curve), and Strength (edge transparency).
  * Smoothly blends tile media directly into the wallpaper background.
* **Drop Shadows:** Ambient lighting shadow with custom tint color, blur radius up to 80px, and opacity control.

---

## 10. State Persistence & `.omniwall` Format

### 10.1 Auto-Session Engine
* OmniWall Pro automatically serializes canvas state to `localStorage` under `omniwall_autosession`.
* On app relaunch, it restores all tiles, playlist queues, current track positions, and wallpaper setups without requiring manual saves.

### 10.2 The `.omniwall` File Standard
Layouts export and import as `.omniwall` files (standard JSON structures).

```json
{
  "version": "6.0",
  "name": "Cyberpunk Neon Study",
  "savedAt": "2026-11-08T16:30:00.000Z",
  "wallpapers": [
    {
      "id": "wp_main",
      "name": "Rainy City",
      "volume": 0.35,
      "speed": 1.0,
      "dim": 0.15,
      "blur": 4,
      "brightness": 95,
      "saturation": 120,
      "pulseOn": false,
      "mirror": { "enabled": true, "reflect": true, "count": 2 }
    }
  ],
  "cards": [
    {
      "id": "card_vid_1731078600_abcd",
      "type": "video",
      "x": 120,
      "y": 80,
      "width": 640,
      "height": 360,
      "zIndex": 105,
      "isLocked": false,
      "loopMode": "playlist",
      "videoFit": "fill",
      "looks": {
        "rounded": 12,
        "strokeOn": true,
        "strokeColor": "#3b82f6",
        "strokeWidth": 2,
        "featherOn": false,
        "shadowOn": true,
        "shadowBlur": 30
      },
      "currentIndex": 2,
      "playlist": [
        { "name": "clip1.mp4", "filePath": "D:/Footage/clip1.mp4" },
        { "name": "clip2.mp4", "filePath": "D:/Footage/clip2.mp4" }
      ]
    }
  ]
}
```

---

## 11. Master Keyboard & Mouse Shortcut Reference

### 11.1 Media Playback & Seeking

| Shortcut | Scope | Action |
| :--- | :--- | :--- |
| `Space` | Selected Tile(s) | Play / Pause video or slideshow playback |
| `Double Click` | Target Tile | Play / Pause clicked tile directly |
| `Right Arrow` | Selected Video(s) | Seek forward **3 seconds** (instant paused sync) |
| `Left Arrow` | Selected Video(s) | Seek backward **3 seconds** (instant paused sync) |
| `Ctrl + Right Arrow` | Selected Video(s) | Seek forward **10 seconds** |
| `Ctrl + Left Arrow` | Selected Video(s) | Seek backward **10 seconds** |
| `Shift + Right Arrow` | Selected Video(s) / Image(s) | Seek forward **60 seconds** / Next Image |
| `Shift + Left Arrow` | Selected Video(s) / Image(s) | Seek backward **60 seconds** / Prev Image |
| `Ctrl + 0` | Selected Video(s) | Instant rewind to start (`00:00`) |
| `Shift + 0` | All Canvas Videos | Rewind all canvas videos to start and pause |
| `Ctrl + P` | All Canvas Videos | Global Play / Pause toggle for all wall videos |
| `M` | Selected Video(s) | Mute / Unmute audio |
| `Ctrl + M` | All Canvas Videos | Mute / Unmute all videos on the canvas |

### 11.2 Playlist Navigation

| Shortcut | Scope | Action |
| :--- | :--- | :--- |
| `Shift + 1` | Selected Tile | Jump to **Next Item** in playlist |
| `Shift + 2` | Selected Tile | Jump to **Previous Item** in playlist |
| `Shift + 3` | Selected Tile | Open / Close the Tile Playlist Drawer |
| `Shift + 4` | Selected Tile | Shuffle / Randomize tile playlist order |

### 11.3 Canvas & Selection

| Shortcut | Action |
| :--- | :--- |
| `Ctrl + A` | Select all visible tiles on canvas |
| `D` | Deselect all tiles |
| `L` | Lock / Unlock selected tile(s) |
| `U` | Unify aspect ratio to match source media |
| `Delete` | Remove selected tile(s) from canvas |
| `Ctrl + Z` | Undo last action |
| `Ctrl + Shift + Z` / `Ctrl + Y` | Redo action |
| `H` | Toggle Clean Presentation Mode (hide all HUDs) |
| `Ctrl + F` | Toggle Borderless Fullscreen |

### 11.4 Universal Audio Player

| Shortcut | Action |
| :--- | :--- |
| `Alt + P` | Play / Pause audio track |
| `Alt + [` | Jump to previous audio track |
| `Alt + ]` | Jump to next audio track |

### 11.5 Layout Management

| Shortcut | Action |
| :--- | :--- |
| `Ctrl + S` | Open Layout Presets Manager |
| `Ctrl + O` | Open file dialog to import `.omniwall` file |

---

## 12. Future Architectural Roadmap & Ideas

Here are potential enhancements planned for future versions of OmniWall Pro:

### 12.1 Real-Time Audio-Reactive Visualizers
* Route the Universal Audio Player's `AudioNode` into custom WebGL/GLSL pixel shaders (frequency spectrum bars, fluid wave reflections, and bass-reactive wallpaper pulsing).

### 12.2 Multi-Display Spanning & Display Pinning
* Add multi-monitor detection, allowing specific wallpaper layers to pin to secondary screens or stretch seamlessly across ultra-wide multi-monitor setups.

### 12.3 Smart Collections & Tag-Based Dynamic Playlists
* Automated folder monitoring that dynamically adds new media dropped into watched system folders without requiring an app reload.
* Tagging system for playlists (e.g., `#Cinematic`, `#Tutorial`, `#Reference`) with instant tag search filtering.

### 12.4 Direct Stream Output (Virtual Camera / OBS NDI)
* Output the active OmniWall canvas directly as an NDI or virtual webcam stream for live video feeds, DJ backdrops, or creative presentations.

## 13. Getting Started & Installation

### Prerequisites
- Node.js (v18+ recommended)
- npm

### Installation & Run
1. Clone the repository:
   ```bash
   git clone https://github.com/DrAmro-s/omniwall-pro.git
   cd omniwall-pro
   ```
2. Install dependencies:
   ```bash
   npm install
   ```
3. Launch the desktop application:
   ```bash
   npm start
   ```

---
## License
Distributed under the MIT License. See `LICENSE` for details.