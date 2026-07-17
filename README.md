# Islam Emad — Electronics & Communications Engineer

A working record of the systems I've built: embedded firmware, real-time signal processing, computer-vision/RF pipelines, and applied AI tooling — plus the courses and certifications behind them.

Every project here was designed, implemented, and debugged end-to-end by me. The write-ups below describe **what each system does, how it's built, and what actually went wrong while building it** — no invented business metrics, no unverifiable ROI numbers. Where a number appears (a latency figure, a baud rate, a buffer size), it's a design parameter or a measurement taken during testing, not a marketing claim.

---

## 📑 Table of Contents

1. [Technical Snapshot](#-technical-snapshot)
2. [Projects](#-projects)
   - [1. Dual-Axis Solar Tracker — ATmega32A](#1-dual-axis-solar-tracker---atmega32a)
   - [2. VoxLink — Low-Latency Mesh Voice over ESP8266](#2-voxlink---low-latency-mesh-voice-over-esp8266)
   - [3. Autonomous Obstacle-Avoidance Rover — ATmega328P](#3-autonomous-obstacle-avoidance-rover---atmega328p)
   - [4. ESP32 Walkie-Talkie with Adaptive Noise Cancellation](#4-esp32-walkie-talkie-with-adaptive-noise-cancellation)
   - [5. Bare-Metal Hospital Management System — C99](#5-bare-metal-hospital-management-system---c99)
   - [6. 24 GHz mmWave Presence & Motion Tracker](#6-24-ghz-mmwave-presence--motion-tracker)
   - [7. Advanced System for Drone Detection, Tracking, and Neutralization](#7-advanced-system-for-drone-detection-tracking-and-neutralization)
   - [8. Autonomous Vulnerability Scanner with AI Triage](#8-autonomous-vulnerability-scanner-with-ai-triage)
   - [9. Bluetooth Smart Home Relay Controller — ATmega32](#9-bluetooth-smart-home-relay-controller---atmega32)
3. [Certifications & Coursework (54 total)](#-certifications--coursework-54-total)
4. [Contact](#-contact)

---

## 🧭 Technical Snapshot

| Domain | Tools / Concepts |
|---|---|
| Embedded / Bare-metal | AVR-GCC, ATmega32 / ATmega32A / ATmega328P, ESP32, ESP8266, register-level programming (Timers, ADC, USART, PWM, ISRs) |
| Real-time DSP | ADPCM, spectral subtraction, NLMS adaptive filtering, FFT/PSD estimation, Kalman filtering |
| Networking | UDP/broadcast mesh, ESP-NOW, UART/Bluetooth SPP, LwIP |
| Computer Vision / Edge AI | YOLO (v8/v10/v11), NCNN, INT8 quantization, NEON SIMD, real-time object tracking (SORT/Hungarian) |
| Applied AI / Automation | Gemini API (few-shot / CoT prompting), asyncio pipelines, multi-agent orchestration |
| Security / Countermeasures | RF spectral analysis, RF countermeasure sweeping, OWASP ZAP, Nmap, Bandit, Semgrep, CVSS scoring |
| Languages | C, C++, Python, Bash |
| Simulation / Tooling | Proteus VSM, Docker, Redis, gRPC, Spectrum Analyzer |

---

## 🚀 Projects

### 1. Dual-Axis Solar Tracker — ATmega32A

**What it does:** Tracks the sun on two axes using four light-dependent resistors (LDRs) instead of GPS/astronomical calculations, keeping a small solar panel closer to perpendicular with the sun throughout the day.

**Why this approach:** Fixed panels lose a significant share of potential output as the sun moves off-axis (irradiance falls off with the cosine of the incidence angle). GPS/ephemeris-based trackers solve this but need a real-time clock, geolocation, and more compute than a simple 8-bit MCU comfortably offers. Instead, four LDRs are arranged in a 2×2 grid behind a cross-shaped physical divider, so each quadrant sees the sun differently depending on its position — this turns "where is the sun" into a simple analog comparison problem.

**How it's built:**
- ATmega32A running bare-metal C in a fixed 20ms control loop (no RTOS).
- The 10-bit ADC samples all four LDR channels sequentially; readings are averaged over 4 samples to cut noise.
- Horizontal and vertical error signals are computed by comparing diagonal pairs of LDRs (e.g., left pair vs. right pair).
- A ±20-count dead-band prevents the servos from constantly hunting for a "perfect" reading that doesn't exist due to ADC noise.
- Two servos are driven via Timer1 Fast-PWM (standard 1000–2000µs pulse range), with step size proportional to the error — bigger correction when far off, finer correction near alignment.

**A real problem I hit:** The servos jittered occasionally even when the error signal was inside the dead-band. Tracing it with a logic analyzer ruled out a PWM/software issue — the pulses were clean. The actual cause was electrical: both servos moving at once drew a current spike that dipped the shared 5V rail, which fed back into the ADC's reference voltage and corrupted the LDR readings, creating a false error that triggered another correction — a small feedback loop of its own. Fixed with three changes: a separate regulator for the servos, a bulk capacitor at the servo power header to absorb the current spikes locally, and averaging multiple ADC samples per reading as a software-side noise filter. It was a good reminder that in mixed analog/digital circuits, power integrity issues can look exactly like software bugs.

**Stack:** `ATmega32A` `Bare-metal C` `Register-Level Programming (ADC & Timer1)` `10-bit ADC (Sequential Sampling)` `Timer1 Fast-PWM (Servo Control)` `Analog Signal Conditioning & Power Integrity` `Hardware De-jittering (Decoupling & Isolation)`

---

### 2. VoxLink — Low-Latency Mesh Voice over ESP8266

**What it does:** Real-time, multi-node voice communication between ESP8266 boards over Wi-Fi, with no router, no server, and no cloud relay — just direct UDP broadcast between nodes.

**Why this approach:** Raw 8kHz/16-bit PCM audio doesn't fit comfortably into an 802.11 mesh with more than one or two active streams, and cloud/WebRTC-style voice relays add 100–300ms of round-trip latency from handshakes and server hops — too much for natural conversation. The fix was to compress the audio on-device to keep the bitrate low, and skip TCP entirely in favor of a loss-tolerant UDP broadcast, accepting the occasional dropped packet in exchange for much lower and more predictable latency.

**How it's built:**
- I2S DMA continuously pulls 20ms audio frames from an INMP441 MEMS microphone into a ring buffer.
- A custom IMA ADPCM encoder compresses each frame roughly 4:1, taking the stream down from 128kbps to about 32kbps.
- Frames go out as small UDP broadcast datagrams (raw sockets, no TCP handshake).
- On the receive side, a small jitter buffer absorbs Wi-Fi timing variance — repeating the last frame on underrun, dropping the oldest on overrun — before decoding back to PCM and pushing it out via I2S DMA to a MAX98357A class-D amp.

**A real problem I hit:** After roughly 45 minutes of continuous full-duplex use, the device would silently freeze. The cause was heap fragmentation: allocating a small buffer on every single DMA interrupt (50 times a second) eventually left the allocator with only unusably small free blocks, and an allocation inside interrupt context eventually failed outright. The fix was to remove dynamic allocation from the hot path entirely — every audio buffer, filter state, and UDP payload buffer became a statically sized global array set at compile time. A related socket-handling leak (control blocks not released when a send briefly failed under contention) was fixed with a small watchdog that force-closes stuck sockets after 500ms.

**Stack:** `ESP8266 (Xtensa LX106)` `Bare-metal C` `IMA ADPCM Compression` `I2S DMA & Ring Buffers` `UDP Broadcast (LwIP Stack)` `INMP441 MEMS Microphone` `MAX98357A Class-D Amp` `Static Memory Allocation (Hot Path)`

---

### 3. Autonomous Obstacle-Avoidance Rover — ATmega328P

**What it does:** A small rover that senses obstacles on three sides (front, left, right) and reroutes or stops using direct register-level control, without an RTOS or Linux-based board in the loop.

**Why this approach:** Software abstraction layers (RTOS schedulers, Arduino's `digitalWrite`/`analogWrite`) add small but real and *variable* delays. For a rover that needs to stop before hitting something, a worst-case delay matters more than an average one, so the whole sense→decide→act path was written directly against the AVR registers to keep timing predictable.

**How it's built:**
- Three HC-SR04 ultrasonic sensors, triggered in round-robin to avoid their pulses interfering with each other.
- Timer/Counter1's input-capture unit measures echo pulse width directly in hardware.
- A small state machine reacts to distance thresholds: under ~20cm triggers an immediate stop/pivot (direct port writes to the H-bridge), under ~40cm halves the PWM duty cycle to slow down, otherwise full speed ahead.
- The full sense-to-motor-response loop is designed to land under ~15ms.

**A real problem I hit:** Using Arduino's `digitalWrite()` for the H-bridge control pins added an inconsistent 8–12µs of overhead per call from its internal lookup-table translation — small on its own, but multiplied across six control pins it created an intermittent 20–30ms delay and, worse, a brief window where the H-bridge could receive a stale direction signal before the new PWM value latched (a real risk of a current "shoot-through" in the driver). Replacing all of those calls with direct `PORTB` bit operations, and updating direction + PWM together inside a single timer interrupt, brought the response down to a consistent sub-5ms and removed the race condition.

**Stack:** `ATmega328P` `Bare-metal C++` `Register-Level Programming (PORTB/D Manipulation)` `Timer1 Input Capture & Hardware PWM` `HC-SR04 Ultrasonic Sensors` `L298N H-bridge Motor Driver` `Deterministic Interrupt Context`

---

### 4. ESP32 Walkie-Talkie with Adaptive Noise Cancellation

**What it does:** Full-duplex, push-to-talk-free voice communication between two ESP32 boards over ESP-NOW, with on-device noise reduction so speech stays intelligible in noisy environments.

**Why this approach:** Analog walkie-talkies have no way to clean up noise, and IP-based voice solutions (Wi-Fi router, cloud relay) reintroduce the 100ms+ latency problem. ESP-NOW is a connectionless 802.11 link with no handshake overhead, which made it a good fit for a tight, low-latency audio pipeline — as long as the noise problem could be solved on-device instead of relying on a clean signal.

**How it's built:**
- I2S DMA frames audio from an INMP441 mic in 20ms chunks, which act as the pipeline's heartbeat.
- Stage one: spectral subtraction — an FFT-based estimate of background noise (updated continuously during silence) is subtracted from the live signal.
- Stage two: a 32-tap NLMS adaptive filter cleans up residual narrowband interference that spectral subtraction alone doesn't catch, with its step size normalized to the signal's own power so it stays stable during sudden loud transients.
- Audio is compressed with µ-law encoding and sent as small ESP-NOW frames; the target end-to-end budget is around 15ms.

**A real problem I hit:** Under sustained full-duplex use, audible "chirp" glitches appeared. The cause was a race condition — the interrupt reading the jitter buffer for playback and the callback writing a newly arrived frame into that same buffer could overlap, producing a torn (half-old, half-new) frame. Rather than use a mutex (which would add latency jitter of its own), I switched to a lock-free triple-buffer scheme: incoming frames are written to an inactive slot, then a single atomic index update marks it "ready," and the playback interrupt only ever reads a fully-written buffer.

**Stack:** `ESP32 (Xtensa LX6)` `I2S DMA` `ESP-NOW` `Digital Signal Processing (FFT / NLMS)` `µ-law Audio Compression` `INMP441 MEMS Microphone` `MAX98357A Class-D Amp` `Lock-Free Concurrency`

---

### 5. Bare-Metal Hospital Management System — C99

**What it does:** A console-based patient/appointment management system with role-based access (admin vs. regular user), built with manual memory management and no external libraries or database.

**Why this approach:** This was an exercise in building a genuinely correct data layer from first principles — linked lists, manual `malloc`/`free`, and access control — without leaning on a database engine or garbage-collected runtime to paper over mistakes.

**How it's built:**
- Each patient is a node in a singly-linked list, and each patient node owns its own nested singly-linked list of appointments (a two-level structure).
- A simple login gate compares credentials and sets a global role flag that gates which functions are reachable (admin can write, regular users can only read/search).
- Search is a straightforward linear scan using `strstr()` for substring matching — no indexing, deliberately kept simple and easy to reason about.

**A real problem I hit:** Deleting a patient who had appointments attached caused a memory leak — the original `delete_patient()` freed the patient struct but never walked and freed the nested appointment list first, so those nodes became orphaned on the heap. I confirmed this by tracking `malloc`/`free` call counts across repeated add/delete cycles and watching the count creep upward. The fix was a proper two-pass deletion: first walk and free every appointment node, then unlink and free the patient node itself.

**Stack:** `C99` `Manual Memory Management (malloc/free)` `Singly-Linked Lists` `Role-Based Access Control` `Data Structures`

---

### 6. 24 GHz mmWave Presence & Motion Tracker

**What it does:** Detects human presence, distance, angle, and speed using a 24 GHz Doppler radar module — including detecting a *stationary* person by their breathing motion — and renders it as a live radar-style display.

**Why this approach:** PIR motion sensors can't tell you distance or speed, and completely miss a person who isn't moving. Cameras solve that but raise privacy concerns and need much more compute. A Doppler radar module sits in between: it can register the tiny (sub-millimeter) motion of a chest rising and falling without capturing any image at all.

**How it's built:**
- The RD-03D radar module does its own on-chip FFT/Doppler processing and outputs parsed distance/angle/speed values over UART at 256000 baud — no heavy DSP needed on the microcontroller side.
- An Arduino Uno R4 parses the UART frames, checks the checksum, and applies a small moving-average smoothing filter to reject noise spikes.
- A Processing sketch reads the resulting CSV stream over USB and draws a polar radar view at ~30 FPS, using cyan for stationary/breathing targets and amber for moving ones.

**A real problem I hit:** Reflections off static metal lab equipment occasionally showed up in the data as small speed spikes (5–10 cm/s) even when nothing was moving — which defeats the whole point of reliably confirming "someone stationary is here." The fix combined a Hamming-windowed smoothing filter with a simple dead-band: any speed reading under 2 cm/s is clamped to zero and the target is flagged as static. That removed the false positives without hiding genuine slow movement.

**Stack:** `24 GHz Doppler radar (RD-03D)` `UART (256000 baud)` `Arduino Uno R4 (C++)` `Processing` `Signal Smoothing (Hamming Window)`

---

### 7. Advanced System for Drone Detection, Tracking, and Neutralization

**What it does:** A hardware-in-the-loop autonomous defense countermeasure system that detects rogue drones via camera frames, tracks them across frames, and — if the detection is confirmed with high confidence over multiple frames — triggers an RF countermeasure sweep across common drone control/GPS/video frequency bands based on RF spectral analysis. **This system was awarded 5th Place for Best Poster at the 43rd National Radio Science Conference (NRSC 2026) and achieved an A+ grade at Horus University.**

**Context:** Developed as a senior graduation capstone project at Horus University. The core engineering objective was exploring how far a real-time detect-and-track pipeline could be pushed on resource-constrained, edge-only hardware (no cloud dependencies, no dedicated GPU workstation), mapping out the exact limits of edge computing in defense applications.

**How it's built:**
- A camera feeds 720p frames directly into a V4L2 DMA buffer to avoid unnecessary memory copies across the pipeline.
- Frames are pre-processed with NEON SIMD intrinsics (handling resizing, color space conversion, and normalization) to keep the ingestion stage highly optimized.
- Inference runs on a quantized (INT8) YOLOv8-nano model exported to NCNN, explicitly compiled and tuned to run efficiently across the Raspberry Pi 5’s CPU cores.
- Detections are tracked frame-to-frame with a Kalman filter (using a constant-velocity model) combined with Intersection-over-Union (IoU) based matching (SORT-style), ensuring a track survives brief physical occlusions and targets drone profiles exclusively.
- A small state machine (CLEAR → DETECTED → CONFIRMED → ENGAGE) requires consecutive high-confidence detections before escalating to countermeasure trigger, preventing single-frame false positives from activating the RF countermeasure sweep.

**A real problem I hit:** Under sustained full-duplex operation and load, the standard Linux scheduler would occasionally preempt the inference thread to handle background OS tasks or interrupt work, causing sudden latency spikes of 15–20ms — which completely blew the pipeline's strict real-time timing budget. I diagnosed this behavior using `perf sched` and `ftrace`. The problem was resolved by isolating specific CPU cores exclusively for the inference thread using `isolcpus` and `irqaffinity`, and changing the thread scheduling policy to real-time (`SCHED_FIFO`) with maximum priority instead of relying on standard sleep/wake time-sharing scheduling. This architectural shift brought timing jitter down from ~20ms to a rock-solid sub-2ms window.

**Stack:** `Raspberry Pi 5 (Debian-based Linux)` `YOLOv8-nano / NCNN` `INT8 quantization` `Kalman filtering` `Real-time Linux scheduling (SCHED_FIFO)` `RF Spectral Analysis & Countermeasures`

---

### 8. Autonomous Vulnerability Scanner with AI Triage

**What it does:** An automated pipeline that runs Nmap and OWASP ZAP against a target, then uses an LLM (Gemini) to cut through the raw findings, filter out irrelevant noise, assign CVSS scores, and generate a prioritized PDF remediation report.

**Why this approach:** Tools like ZAP are thorough but noisy — a scan against a typical single-page app can produce 150–400 alerts, many of which aren't actually exploitable in context. Normally a human has to manually triage that list, which takes real time. The idea here was to use an LLM as a structured triage step: given the raw alert plus context about the target's stack, it classifies relevance and severity in a consistent, schema-enforced format instead of free text.

**How it's built:**
- A Python `asyncio` pipeline runs recon (Nmap fingerprinting), static analysis (Bandit/Semgrep on any available source), and dynamic scanning (OWASP ZAP via its REST API) as separate stages connected through Redis Streams.
- ZAP's alerts are sent to Gemini with a few-shot, chain-of-thought prompt that forces a strict JSON output: CVSS v3.1 vector, CWE/CVE mapping, and a concrete remediation suggestion.
- A report-generation stage turns that structured JSON into a CVSS-ranked PDF using ReportLab.
- The whole thing runs in a multi-stage Docker container with a read-only root filesystem.

**A real problem I hit:** About 30% of runs would fail on a "connection refused" error when the pipeline tried to talk to the ZAP daemon's REST API right after starting it. The cause was a missing readiness check — the ZAP Java process was still starting up (JVM class loading, GC init) when the client tried to connect. Replacing a naive `sleep(10)` with an actual TCP health-check loop (retrying the connection with backoff until ZAP's port is actually accepting connections) took the failure rate from ~30% to 0%.

**Stack:** `Python asyncio` `OWASP ZAP` `Nmap` `Gemini API (few-shot / CoT prompting)` `Docker` `Redis Streams`

---

### 9. Bluetooth Smart Home Relay Controller — ATmega32

**What it does:** Wireless on/off control of four mains-powered devices (lamp, fan, AC, auxiliary) from an Android app over Bluetooth, using an ATmega32 and relays — built and validated as an ITI embedded systems capstone project.

**Why this approach:** The brief specifically ruled out cloud services, Wi-Fi stacks, or an OS — the goal was a fully bare-metal, interrupt-driven design, validated completely in Proteus simulation before any physical wiring.

**How it's built:**
- An Android app sends single-byte ASCII commands over Bluetooth Classic (SPP) to an HC-05 module acting as a transparent UART bridge.
- A USART receive interrupt on the ATmega32 reads each incoming byte and maps it directly to a relay output — no polling loop involved.
- A ULN2003 Darlington array sits between the 5V logic and the relay coils, providing both the current the relays need and electrical isolation (plus flyback protection) between the logic and the 220V AC side.
- End-to-end, tap-to-relay-click latency lands around 50–80ms.

**A real problem I hit:** During Proteus simulation, sending commands in quick succession sometimes caused a relay to switch the wrong channel. The cause was a classic race condition: the receive interrupt and the main loop were both touching the same shared `relay_state` variable without synchronization, so the main loop could read a value mid-update. The fix was a clear ownership model — the interrupt is the only writer of the raw received byte, the main loop is the only consumer, and a simple "command pending" flag hands control between them without needing to disable interrupts globally.

**Stack:** `ATmega32` `Bare-metal C` `USART interrupt` `HC-05 Bluetooth SPP` `ULN2003 relay driving`

---

## 🎓 Certifications & Coursework (54 total)

Organized by issuing platform. Dates are shown as recorded on the certificate ("N/A" where the certificate doesn't include a date).

### Amazon Web Services (AWS)
| Certificate | Date |
|---|---|
| AWS Technical Essentials | Oct 02, 2025 |
| AWS Certified Advanced Networking | Oct 10, 2025 |

### NVIDIA
| Certificate | Focus Area |
|---|---|
| Ansible Essentials for Network Engineers | Ansible playbooks, Jinja2 templating, YAML inventories |
| NVIDIA License System (NLS) — 2025 | License server config, GPU software asset management, HA design |
| NVIDIA License System (NLS) — 2024 | License administration, entitlement enforcement, reporting |
| Cumulus Linux Administration | NVUE object model, VXLAN/EVPN, Linux switchdev |
| From Basics to GenAI Practice | Transformer architectures, diffusion models, quantization, prompt engineering |

### Frontend Masters (28 courses)
| Course | Focus Area |
|---|---|
| Backend System Design | CQRS, event-driven architecture, load balancing |
| Fullstack v3 | REST API design, client-server data flow, DB integration |
| AI Engineering | RAG, LLM evaluation, agentic workflow design |
| JavaScript: The Hard Parts v3 | Closures, prototype chain, event loop |
| Complete Intro to Linux & the Command Line | Bash scripting, filesystem hierarchy |
| Everything Git | Git object model, rebase, reflog recovery |
| Enterprise DevOps | Terraform, CI/CD design, cloud architecture, observability |
| Agent Harness | Agent orchestration, tool-calling, state locking |
| AI-Powered Python | HuggingFace Transformers, PyTorch, fine-tuning |
| Complete Intro to Cloud Infrastructure | Terraform, EC2 autoscaling, ALB, Route 53 |
| The Hard Parts of AI | Backpropagation, gradient descent, computational graphs |
| Computer Science Fundamentals v2 | Big O, recursion, sorting |
| Algorithms | Big O, tree traversals, dynamic programming |
| Complete Intro to Containers v2 | Dockerfile optimization, namespaces, cgroups |
| Node.js v3 | Event loop, CommonJS, async error handling |
| Claude Code | Terminal-based AI tooling, context management |
| Prompt Engineering | Few-shot & chain-of-thought prompting, token optimization |
| DevOps | CI/CD, Docker, IaC, automated testing |
| Practical Algorithms | Dynamic programming, divide & conquer, greedy algorithms |
| Complete Intro to Go | Goroutines, channels, interface composition, table-driven tests |
| TensorFlow.js | WebGL-accelerated inference, client-side fine-tuning |
| Pro AI Workflows | Cursor IDE, Claude Code CLI, prompt engineering |
| Model Context Protocol (MCP) | JSON-RPC 2.0, client-server transport, tool primitives |
| Practical Python | — |
| TypeScript, Go & Rust | — |
| Backend Architecture | — |
| Deep JavaScript v3 | — |
| AI Agents v2 | — |

### Mahara-Tech (ITI, 7 courses)
| Course | Date |
|---|---|
| Computer Network Fundamentals | 03/10/25 |
| Introduction to Network Security | 06/10/25 |
| Introduction to Deep Learning | 23/01/26 |
| Python Programming Basics | N/A |
| Creating Declarative User Interfaces | 02/10/25 |
| CMOS Fabrication Fundamentals | 08/10/25 |
| Freelancing Basics | 31/10/25 |

### OpenLearn (The Open University)
| Course | Date |
|---|---|
| Introducing Engineering | Oct 7, 2025 |
| Exploring Communications Technology | Oct 13, 2025 |
| Digital Communications | Oct 22, 2025 |
| Network Security | Oct 15, 2025 |

### Other Platforms
| Certificate | Issuer | Notes |
|---|---|---|
| Managing a Business | ITI (Mahara-Tech) | Business institute track |
| Introduction to Nephio (LFS179) | The Linux Foundation | Oct 07, 2025 |
| Companies Establishment | EEIC | Entrepreneurship track |
| HUE Certificate | ITI (Mahara-Tech) | — |
| Islam Emad Ahmed — Certificate | HUE | — |
| Artificial Intelligence for Business Professionals | HP LIFE Academy | — |
| Neural Networks Certificate | WE | TensorFlow/Keras, supervised learning fundamentals |
| Google — Certificate | ITI (Mahara-Tech) | Prompt engineering, LLM API integration, RAG, cloud AI services |
---

## 📬 Contact

Open to discussing any of these projects in more depth — architecture decisions, register-level optimization, firmware design, or the reasoning behind specific engineering trade-offs. 

Feel free to reach out directly:

* **Email:** [islamemad811@gmail.com](mailto:islamemad811@gmail.com)
* **LinkedIn:** [linkedin.com/in/islam-ismail-18a88b381](https://www.linkedin.com/in/islam-ismail-18a88b381)
* **GitHub:** [github.com/islam-8](https://github.com/islam-8)
* **Phone:** +20 155 993 3466
* **Location:** Mansoura, Dakahlia, Egypt
