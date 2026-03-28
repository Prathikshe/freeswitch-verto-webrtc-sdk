# 📞 Verto WebRTC SDK + FreeSWITCH Setup Guide

## 📌 Overview

The **mod_verto** module in FreeSWITCH enables **WebRTC-based communication** using **WebSockets (WS/WSS)**.

It allows browser clients to:
- Register SIP users
- Make / receive calls
- Control call states (hold, transfer, etc.)

This guide covers:
- FreeSWITCH configuration (`verto.conf.xml`)
- User directory setup (`complete_directory.xml`)
- Dialplan routing
- Frontend SDK integration (`verto-sdk.js`)

---

# 🏗️ Architecture

```text
Browser (Verto SDK)
        ↓
WebSocket (WS/WSS)
        ↓
FreeSWITCH (mod_verto)
        ↓
SIP / Dialplan / Agents
