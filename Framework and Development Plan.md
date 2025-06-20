


# Framework and Development Plan

This document describes the framework selection, development strategy, and deployment process for the CNC HMI system that is developed on a PC and deployed to run on a Raspberry Pi (Pi 3/4 or CM4).

---

## 📦 Framework: LVGL (Light and Versatile Graphics Library)

**LVGL** is a cross-platform GUI library designed for embedded and low-power systems. It is selected due to its:

- 🟢 **Extremely lightweight footprint** — ideal for Raspberry Pi
- 🟢 **Runs directly on the framebuffer** (`/dev/fb0`), no desktop/X11 required
- 🟢 **Full touch support** with resistive/capacitive screens
- 🟢 **Widget-rich, scalable UI** — buttons, lists, tables, and custom graphics
- 🟢 **Written in C**, easily integrated with low-level serial logic

---

## 💻 Development Environment

| Feature | Details |
|--------|---------|
| **Host** | Linux PC (Ubuntu/Debian recommended) |
| **Framework** | LVGL + Linux framebuffer port |
| **Editor/IDE** | VS Code / CLion / Qt Creator |
| **Build System** | `Make` or `CMake` |
| **Cross-Testing** | Local framebuffer emulation, then hardware-in-loop |
| **Version Control** | Git (GitHub repo) |

---

## 🍓 Deployment Environment

| Target | Raspberry Pi (Model 3B/4/CM4) |
|--------|-------------------------------|
| OS | Raspberry Pi OS Lite or Full |
| Display | Official 7" Pi Touchscreen or HDMI |
| Input | Touch (via `/dev/input/eventX`) |
| Serial | USB to RS232/RS485 converters or UART GPIOs |
| GUI Mode | Direct framebuffer (`/dev/fb0`) — no desktop required |
| Boot Mode | Headless auto-start into GUI via systemd |

---

## 🧱 Development Strategy

### 1. 🖥️ **PC Development Flow**
- Code and test using LVGL’s **Linux framebuffer port** on desktop
- Simulate serial data if hardware is not connected
- Use modular C/C++ structure for easy portability

### 2. 🧪 **Test and Debug on PC**
- Stub/mock serial functions for unit testing
- Log data to files to simulate CNC CPU and APU responses

### 3. 🍓 **Transfer to Raspberry Pi**
- Push source code via Git or SCP
- Install build tools and LVGL dependencies on Pi
- Compile and run on Pi's framebuffer
- Connect RS232 and RS485 via USB converters or GPIO-UART

---

## 🔁 Development Workflow

```plaintext
Design LVGL UI → Test on PC → Integrate serial modules →
Simulate CNC data → Deploy to Pi → Field test on real hardware
````

---

## ⚙️ Raspberry Pi Setup Steps

### 🧩 Dependencies

```bash
sudo apt update
sudo apt install git make g++ libinput-dev libevdev-dev libudev-dev libdrm-dev
```

### 📦 Clone and Build

```bash
git clone https://github.com/lvgl/lv_port_linux_frame_buffer.git
cd lv_port_linux_frame_buffer
make
```

### 🖥️ Run on Framebuffer

```bash
sudo ./demo
```

---

## 🔌 Serial Communication Design

| Interface | Role   | Direction | Purpose                        |
| --------- | ------ | --------- | ------------------------------ |
| **RS232** | Slave  | CPU → HMI | Counter updates, file requests |
| **RS485** | Master | HMI → APU | Axis polling and data updates  |

**Note:** CRC validation and retry logic will be implemented in both modules.

---

## 🚀 Deployment Plan (Raspberry Pi)

### 1. **Deploy Binary or Source**

* Option A: Compile on Pi using same source
* Option B: Cross-compile on PC and SCP to Pi

### 2. **Autostart GUI**

Create a systemd service `/etc/systemd/system/hmi.service`:

```ini
[Unit]
Description=CNC HMI
After=multi-user.target

[Service]
ExecStart=/home/pi/hmi/hmi_app
WorkingDirectory=/home/pi/hmi
StandardOutput=inherit
StandardError=inherit
Restart=always
User=pi

[Install]
WantedBy=multi-user.target
```

Enable with:

```bash
sudo systemctl enable hmi.service
```

### 3. **Run Fullscreen**

* LVGL will natively occupy the entire display using `/dev/fb0`

---

## 🧰 Folder Structure Plan

```
hmi/
├── main.c
├── gui/
│   └── screen_layout.c
├── serial/
│   ├── rs232_handler.c
│   └── rs485_handler.c
├── utils/
│   └── crc.c
├── assets/
│   └── fonts, images
├── logs/
│   └── runtime.log
├── Makefile
└── README.md
```

---

## ✅ Summary

| Stage                 | Status                                                        |
| --------------------- | ------------------------------------------------------------- |
| **Framework Chosen**  | LVGL (C, framebuffer-based)                                   |
| **Development on PC** | Testable with simulated serial data                           |
| **Deployment on Pi**  | Headless GUI, systemd startup, touch support                  |
| **GUI Features**      | Axis live position, counter display, file upload, status logs |
| **Serial Support**    | RS232 slave (CPU), RS485 master (APU)                         |
| **Goal**              | Lightweight, responsive, production-ready CNC HMI             |

---

*Maintained by Adarsh Gajula | June 2025*

```

---

Let me know if you'd like a separate `system-architecture.md` or `protocol.md` document next, or if you'd like this split into smaller sections like `docs/` folder on GitHub.
```
