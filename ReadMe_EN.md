# Windows 11 ARM64 Clean Install Recovery

Technical guide for the **bare-metal** reinstallation of **Windows 11 ARM64** on **Samsung Galaxy Book (Go / Edge / S)** devices with ARM architecture.

---

## Description

This project provides a comprehensive guide for the **complete recovery** of Windows 11 ARM64 in scenarios where:

- The **UFS drive is corrupted, replaced, or formatted**.
- The standard Windows installer fails to complete the installation.
- A **bare-metal** deployment is required via **WinPE and DISM**.

This procedure is designed for technicians and advanced users, documenting every step: ADK environment preparation, driver injection, WinPE creation, and final setup.

---

## Prerequisites

- **Host PC**: Windows 10/11 x64
- **OS Image**: Official **Windows 11 ARM64** ISO
- **Tools**: **Windows ADK** + **WinPE add-on**
- **Drivers**: Original drivers from the [Samsung Support](https://www.samsung.com/support) website

---

## Quick Start

1.  **Clone or download this repository:**
    ```bash
    git clone [https://github.com/](https://github.com/)<your-username>/galaxybook-arm64-recovery.git
    ```
2.  **Open the guide:**
    ```text
    Windows11_ARM64_BareMetal_Restore_Samsung.md
    ```
3.  **Follow the step-by-step instructions.**

---

## Disclaimer

> **Warning:** This guide is intended for experienced technical personnel. Improper use may result in data loss or permanent damage to the device.

---

## License

Distributed under the **MIT License** — see the [`LICENSE`](LICENSE) file for details.

---

© 2026 Flavio