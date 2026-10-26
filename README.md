# 🚀 TDesktop Clean: Patched Telegram Desktop

[![Build Patched Telegram Desktop](https://github.com/Dantiox/tdesktop-clean/actions/workflows/build.yml/badge.svg)](https://github.com/Dantiox/tdesktop-clean/actions/workflows/build.yml)
[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

A lightweight, high-performance, and 100% transparent patchset for official **Telegram Desktop** (`tdesktop`), designed for Linux power users, battery efficiency, and zero telemetry.

---

## ⚡ Why this fork?

Popular third-party clients (such as AyuGram and freshGram) solve two simple problems — **Ghost Mode** and **Anti-Recall** — in an egregiously inefficient manner:
* **88 Concurrent Threads:** Excessive thread spawning competing for the Linux scheduler.
* **Continuous Background I/O:** Encrypted SQLCipher database transactions logging every incoming event across all public channels.
* **Battery Drain:** CPU never enters deep low-power sleep states (**Package C6/C8 on Zen 4 / Ryzen**), cutting laptop battery life from 10 hours down to 3.5 hours.
* **Closed Components:** Controversy surrounding private submodules and obfuscated hooks in parts of the ecosystem.

**TDesktop Clean** takes the official upstream Telegram Desktop and applies a minimal, surgical in-memory patch:
* **0 Extra Threads:** Uses Telegram's native `crl` thread pool.
* **0 Background Disk Thrashing:** Preserves recalled messages directly in the session's memory model.
* **0.0% CPU Idle:** Lets modern laptop CPUs stay in deep sleep.
* **100% Auditable Open Source (GPLv3).**

---

## ✨ Features

1. **👻 Ghost Mode (Invisibility):**
   * Blocks outgoing `messages.readHistory` MTProto RPCs. You read messages locally while the sender still sees 1 checkmark.
   * Blocks `messages.setTyping` RPCs. No typing indicator is ever broadcast.
2. **🛡️ Anti-Recall (Save Deleted Messages):**
   * Intercepts `HistoryItem::destroy()`. When an incoming message is revoked by the sender, it is not destroyed.
   * The message remains visible with a `[Удалено]` prefix.
3. **📝 Anti-Edit (Edit History):**
   * Preserves previous versions of edited messages in an in-memory revision vector.

---

## 📦 Installation via Flatpak

Pre-built standalone bundles are automatically compiled in the cloud via GitHub Actions:

1. Go to the [Actions Tab](https://github.com/Dantiox/tdesktop-clean/actions).
2. Click the latest successful workflow run and download the `telegram-clean-flatpak` artifact.
3. Unzip and install with a single command:
   ```bash
   flatpak install --user telegram-clean.flatpak
   ```

---

## 🛠️ Building Locally

If you prefer building locally:
```bash
flatpak-builder --user --install --force-clean build-dir org.telegram.desktop.yml
```

## 📜 License
Licensed under the GNU General Public License v3.0 (GPL-3.0). Upstream source code copyright Telegram FZ-LLC and contributors.
