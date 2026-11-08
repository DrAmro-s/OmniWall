# OmniWall Pro — Master Edition

A high-performance, GPU-composited desktop media dashboard and multi-layer reactive live wallpaper engine running in Electron / Chromium. 

Built as a self-contained, zero-dependency single-file frontend (`OmniWall.html`) designed for zero-latency local playback, high-bitrate streaming, and seamless state persistence.

---

## Key Architecture & Capabilities

### 1. Multi-Tile Media Canvas
- **Dual-Buffer Video Tiles:** Alternating `vidA` / `vidB` HTML5 elements delivering seamless 0 ms gapless video looping and automated playlist sequencing without black frames.
- **Dynamic Image Slideshows:** In-memory Blob streaming engine bypassing local file sandbox constraints, featuring 4 framing modes (Zoom In, Zoom Out, Tile Grow, Static) with crossfades down to 0.05s.
- **Docked Web Tiles:** Dedicated HUD toggling between Drag/Move and native interaction modes with Piped/Invidious embed fallback for restricted streams.
- **Visual Styling Engine:** Dynamic border curvature, custom stroke outlines, drop shadows, and elliptical/rectangular alpha edge feathering.

### 2. Multi-Layer Reactive Live Wallpaper
- **Multi-Tab Profiles:** Color-coded layer stacks with independent video, image, or stream profiles.
- **60 FPS Hardware Canvas Mirror Wings:** Center-anchored reflection wings mirroring edge video pixels via `requestVideoFrameCallback` to eliminate widescreen letterbox bars.
- **Smart Aspect Ratio Handling:** Automatic portrait (9:16) detection preventing vertical videos from zooming and cropping.
- **Interactive Gear Mode:** Real-time z-index elevation enabling native YouTube 1080p/4K resolution selection directly on live wallpaper streams.

### 3. Universal Audio Player Suite
- **Perceptual 200% Gain Stage:** Integrated Web Audio API node tree delivering clean volume amplification beyond system defaults.
- **Synchronized Video Peek:** Side-by-side drawer displaying video peek monitors for audio-video files alongside queue search and shuffle algorithms.
- **Auto-Retracting Glass Drawer:** 6-second idle detector with GPU motion-blur retraction.

### 4. Apple VisionOS Design System
- Multi-stop linear glass gradients with hardware-accelerated `backdrop-filter: blur(38px) saturate(190%)`.
- Hairline specular edge lighting with isolated composite rendering layers.

---

## Getting Started

### Prerequisites
- Node.js (v18+ recommended)
- npm

### Installation & Run
1. Clone the repository:
   ```bash
   git clone [https://github.com/DrAmro-s/omniwall-pro.git](https://github.com/DrAmro-s/omniwall-pro.git)
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

## Keyboard Controls Overview

| Key | Action |
| :--- | :--- |
| `Space` | Play / Pause selected tile |
| `Double Click` | Play / Pause clicked tile |
| `U` | Unify tile aspect ratio to source dimensions |
| `L` | Lock / Unlock tile position and sizing |
| `D` | Deselect all active tiles |
| `M` / `Ctrl + M` | Mute selected / Mute all video tiles |
| `Ctrl + P` | Play / Pause all wall videos |
| `Ctrl + S` / `Ctrl + O` | Layouts Manager / Import `.omniwall` file |
| `Shift + 1` / `Shift + 2` | Next / Previous track in tile playlist |
| `Alt + P` / `Alt + [` / `Alt + ]` | Universal Audio Player transport controls |
| `H` | Toggle clean presentation mode (hide all UI) |
| `Ctrl + F` | Toggle borderless fullscreen |

---

## License
Distributed under the MIT License. See `LICENSE` for details.