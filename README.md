<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║          ISLAM EMAD — SYSTEMS ENGINEER PORTFOLIO             ║
║     Edge AI · Embedded Systems · 5G/6G · Cybersecurity      ║
╚══════════════════════════════════════════════════════════════╝
```

[![Status](https://img.shields.io/badge/Status-Seeking_2026-68d391?style=flat-square&labelColor=0b0f1a)](https://islam-8.github.io/portfolio)
[![Domain](https://img.shields.io/badge/Domain-Aerospace_%26_Defense-63b3ed?style=flat-square&labelColor=0b0f1a)](https://islam-8.github.io/portfolio)
[![University](https://img.shields.io/badge/University-Horus_University_ECE-b794f4?style=flat-square&labelColor=0b0f1a)](https://horus.edu.eg)
[![Certs](https://img.shields.io/badge/Certifications-50%2B-f6ad55?style=flat-square&labelColor=0b0f1a)](https://islam-8.github.io/portfolio#certs)

**[🌐 Live Portfolio](https://islam-8.github.io/portfolio)** · **[💼 LinkedIn](https://www.linkedin.com/in/islam-ismail-18a88b381)** · **[💬 WhatsApp](https://wa.me/201559933466)**

</div>

---

## 🧠 About

Systems Engineer graduating 2026 from Horus University (B.Sc. Communications & Electronics). I build systems that perform under **real-world constraints** — squeezing AI pipelines onto a $100 SBC, writing bare-metal C for microcontrollers, designing RF security frameworks, and architecting multi-agent AI pipelines from scratch.

```
42 FPS   · Sentinel idle detection mode (Raspberry Pi 5)
<40 ms   · Full AI pipeline loop latency
88.8%    · YOLOv8-nano mAP50 on drone dataset
1.2 km   · RF spectrum monitoring range (HTOOL-SA 6G)
50+      · Completed certifications (NVIDIA, AWS, Linux Foundation, ITI)
```

---

## 🚀 Featured Projects

### 🛡 Horus Sentinel — C-UAS Platform *(Graduation Project — Team Leader)*
> Counter-drone detection, tracking & neutralization system. Submitted to **NRSC 2026**.

| Metric | Value |
|--------|-------|
| Detection FPS | 42 fps (idle) · 28 fps (detect) |
| Loop Latency | < 40 ms end-to-end |
| Model | YOLOv8-nano · mAP50 = 88.8% |
| RF Range | 1.2 km (HTOOL-SA 6G SDR) |
| Codebase | 24 Python files · ~4,800 lines |

**Pipeline:** `MOG2 → YOLOv8-nano (frame-skip=2) → CSRT+Kalman tracking → PTZ stepper slew → RF spectrum → Flask+SocketIO dashboard → Telegram alerts`

**Stack:** `Python` `OpenCV` `YOLOv8` `Kalman Filter` `Flask` `SocketIO` `GPIO` `SQLite` `Raspberry Pi 5`

**Supervised by:** Prof. Dr. Amr Hussein · Assoc. Prof. Dr. Mervat Elseddek · Dr. Rania Al-Abd

---

### 🔐 Automated Vulnerability Scanner *(5-Agent AI Pipeline)*
> AI-powered security audit platform with end-to-end agent orchestration.

**Agent Pipeline:**
```
Recon Agent (Nmap/Shodan)
    └→ Static Analysis (semgrep/bandit)
        └→ Dynamic Testing (OWASP ZAP)
            └→ Validation (EPSS/CVSS scoring)
                └→ Report Agent (PDF/Jira/Telegram)
```

**Result:** CVSS-ranked vulnerability PDF generated in < 60 seconds

**Stack:** `Python` `FastAPI` `Gemini AI` `Docker` `OWASP ZAP` `Nmap` `Shodan`

---

### 📡 Digital Walkie-Talkie — DSP Noise Cancellation
> Full-duplex P2P voice over ESP-NOW. No router needed.

**DSP Chain:** `I2S Mic → Spectral Subtraction + NLMS Adaptive Filter → I2S Amplifier`

| Metric | Value |
|--------|-------|
| Protocol | ESP-NOW P2P (~4 ms TX latency) |
| Pipeline | ~15 ms total latency |
| SNR Gain | ~12 dB improvement |

**Stack:** `ESP32` `ESP-NOW` `I2S` `Spectral Subtraction` `NLMS` `DSP`

---

### 🤖 Autonomous Obstacle-Avoiding Rover
> HC-SR04 tri-sensor priority routing algorithm on Arduino.

**Stack:** `Arduino` `HC-SR04` `L298N` `PWM` `C++` · Reroutes in ~15 ms from detection

---

### 📶 ESP8266 WiFi Security Analyzer
> Standalone embedded platform for 802.11 protocol education.

**Hardware:** ESP8266 NodeMCU + SSD1306 OLED (128×64, I2C) + 3-button tactile HMI

**Evolution:** Breadboard prototype → Custom PCB → Li-Po integrated standalone unit

**Stack:** `ESP8266` `Arduino C++` `I2C` `SSD1306` `802.11 Management Frames` `PCB Design`

> ⚠ For Educational & Cybersecurity Awareness Purposes Only

---

### ☀️ LDR Dual-Axis Solar Tracker
> Autonomous solar tracking with LDR differential sensing and SolidWorks mechanical design.

**System:** 4× LDR sensors → Arduino Nano → 2× Micro Servo Motors → 3D-printed mount (SolidWorks)

**Stack:** `Arduino Nano` `LDR` `Servo PWM` `SolidWorks` `3D Printing` `C++`

---

## 🛠 Technical Stack

```
EMBEDDED / HARDWARE
  ├── Languages    : C (Advanced) · C++ · Assembly (ARM64)
  ├── MCUs         : STM32 · AVR · ATmega32 · Arduino · ESP32 · RPi 5
  ├── Protocols    : UART · SPI · I2C · CAN · GPIO · PWM
  ├── Tools        : Proteus · SolidWorks · KiCad
  └── RTOS         : FreeRTOS · Bare-metal

AI / COMPUTER VISION
  ├── Frameworks   : PyTorch · TensorFlow · OpenCV
  ├── Models       : YOLOv8 · CSRT · Kalman Filter · NLMS
  ├── Inference    : Edge AI · ARM64 optimization
  └── GenAI        : Gemini API · LLM Agents · Prompt Engineering

WIRELESS / RF
  ├── Standards    : 802.11 b/g/n · 5G NR · 6G concepts
  ├── Hardware     : HTOOL-SA 6G SDR · ESP8266 · ESP32
  └── Analysis     : Spectrum monitoring · Packet analysis

CYBERSECURITY
  ├── Tools        : Nmap · OWASP ZAP · Shodan · semgrep · bandit
  ├── Standards    : CVSS · EPSS · OWASP Top 10
  └── Frameworks   : CVE analysis · Vulnerability scanning

CLOUD / DEVOPS
  ├── Platforms    : AWS Networking · Docker · K8s
  ├── Tools        : Ansible · Nephio · Cumulus Linux
  └── CI/CD        : GitHub Actions · Linux CLI

SOFTWARE
  ├── Python       : Advanced (Flask · SocketIO · FastAPI · asyncio)
  ├── Java         : OOP · REST APIs · RBAC
  └── Algorithms   : DSA · System Design
```

---

## 📜 Certifications (50+)

| Issuer | Notable Certifications |
|--------|----------------------|
| 🟢 **NVIDIA Academy** | Building LLM Apps with Prompt Engineering · AI for All: GenAI · Cumulus Linux Administration · Ansible Essentials |
| 🟠 **AWS Training** | AWS Advanced Networking Specialty Prep · AWS Technical Essentials |
| ⚪ **Linux Foundation** | Introduction to Nephio (LFS179) |
| 🟣 **Frontend Masters** | Complete Intro to CS · Algorithms · AI Agents · Practical Prompt Engineering · Full Stack Fundamentals |
| 🔵 **ITI Egypt** | Embedded Systems Trainee — 162h Intensive (Jul–Sep 2024) |

---

## 🎓 Education

**B.Sc. Communications & Electronics Engineering**
Horus University in Egypt — Faculty of Engineering
*Class of 2026 · Mansoura, Egypt*

---

## 📫 Contact

| Channel | Link |
|---------|------|
| 🌐 Portfolio | [islam-8.github.io/portfolio](https://islam-8.github.io/portfolio) |
| 💼 LinkedIn | [Islam Ismail](https://www.linkedin.com/in/islam-ismail-18a88b381) |
| ⚙️ GitHub | [github.com/islam-8](https://github.com/islam-8) |
| 💬 WhatsApp | [+20 155 993 3466](https://wa.me/201559933466) |

---

<div align="center">

*"I build systems that work under real-world constraints — squeezing AI pipelines onto a $100 SBC, writing bare-metal C for microcontrollers, and designing security frameworks from scratch."*

**Islam Emad · Systems Engineer · Class 2026**

</div>
