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

```
Browser (Verto SDK)
        ↓
WebSocket (WS/WSS)
        ↓
FreeSWITCH (mod_verto)
        ↓
SIP / Dialplan / Agents
```
## ⚙️ Step 1: Configure `verto.conf.xml`

📍 **Path:**

---

### ✅ Key Configurations

- Enable WebSocket (**WS/WSS**)
- Set ports:
  - `8081` → WS  
  - `8082` → WSS  
- Configure SSL (`wss.pem`)
- Enable authentication
- Configure RTP (media) settings
- Configure codecs (`opus`, `vp8`)

---

### 📄 Sample Configuration

```xml
<configuration name="verto.conf" description="HTML5 Verto Endpoint">

  <settings>
    <param name="debug" value="1"/>
    <param name="jsonrpc-port" value="8081"/>
    <param name="bind-params" value="transport=ws,tls"/>
    <param name="auth-calls" value="true"/>
  </settings>

  <profiles>
    <profile name="default">

      <param name="event-channel" value="verto"/>

      <!-- WebSocket Bind -->
      <param name="bind-local" value="172.31.6.223:8081"/>
      <param name="bind-local" value="172.31.6.223:8082" secure="true"/>

      <!-- SSL -->
      <param name="secure-combined" value="$${certs_dir}/wss.pem"/>
      <param name="secure-chain" value="$${certs_dir}/wss.pem"/>

      <!-- Authentication -->
      <param name="userauth" value="true"/>
      <param name="blind-reg" value="false"/>

      <!-- RTP -->
      <param name="rtp-ip" value="172.31.6.223"/>
      <param name="ext-rtp-ip" value="98.130.17.185"/>
      <param name="rtp-start-port" value="16384"/>
      <param name="rtp-end-port" value="32768"/>

      <!-- Codecs -->
      <param name="outbound-codec-string" value="opus,vp8"/>
      <param name="inbound-codec-string" value="opus,vp8"/>

    </profile>
  </profiles>
</configuration>
```
# 👤 Step 2: Configure Users (Directory)

📍 **Path:**

---

## ✅ Required Additions

Add the following parameters to enable Verto communication:

```xml
<param name="jsonrpc-allowed-methods" value="verto"/>
<param name="jsonrpc-allowed-event-channels"
       value="subscribe,broadcast,answer,attach,bye,invite,modify,punt,info,demo,conference,presence"/>
<user id="1001">
  <params>
    <param name="password" value="secret123"/>
    <param name="jsonrpc-allowed-methods" value="verto"/>
    <param name="jsonrpc-allowed-event-channels"
           value="subscribe,broadcast,answer,attach,bye,invite,modify,punt,info,demo,conference,presence"/>
  </params>
</user>
```
## 🔀 Dialplan Routing

Calls are bridged using:

```bash
${verto_contact ${dialed_extension}@domain} | sofia/internal/${agenturi ${dialed_extension} domain}
```
# 🚀 Frontend SDK (verto-sdk.js)

## 📦 Quick Start

### 1. Include Dependencies

```html
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.4.0/jquery.min.js"></script>
<script src="js/verto-sdk.js"></script>
```
## 2. Add Audio Element
```<audio id="webcam" autoplay></audio>```
## 3. Configure SDK
```
VertoPhone.configure({
    socketUrl: 'wss://your-server.com:8082/'
});
```

## 🔐 Login (No Provisioning API)

```javascript
VertoPhone.login({
    agentId: '1001',
    domain: 'sip.yourserver.com',
    password: 'secret123',
    displayName: 'Agent 1001',

    statusList: [
        { status_text: 'Available', status_number: '1000' },
        { status_text: 'Busy', status_number: '1001' },
        { status_text: 'Logout', status_number: '9999' }
    ]
});
```

# VertoPhone SDK Documentation

## 🎧 Event Handlers

```javascript
VertoPhone.setHandlers({

    onConnected: function() {
        console.log('✅ Connected');
    },

    onDisconnected: function() {
        console.log('❌ Disconnected');
    },

    onLoginError: function() {
        console.log('⚠️ Login failed');
    },

    onIncomingCall: function(name, number, dialog) {
        console.log(`📞 Incoming: ${name} (${number})`);
    },

    onCallTrying: function(number) {
        console.log(`📲 Calling ${number}`);
    },

    onCallActive: function(number) {
        console.log(`✅ Active call with ${number}`);
    },

    onCallHangup: function(cause) {
        console.log(`❌ Call ended: ${cause}`);
    },

    onCallHeld: function() {
        console.log('⏸ Call on hold');
    }
});
```

## 📞 Call Controls

```javascript
VertoPhone.call('1002');     // Make call
VertoPhone.hangup();         // Hangup
VertoPhone.answer();         // Answer
VertoPhone.reject();         // Reject
VertoPhone.hold();           // Hold
VertoPhone.transfer('1003'); // Transfer
VertoPhone.dtmf('1');        // Send DTMF
```

## ⚠️ Important Notes

### 🔐 Security

- Credentials are now passed directly (no provisioning API)
- **Avoid exposing raw passwords in production**

**Recommended:**
- Use JWT / short-lived tokens
- Or backend proxy

### 🌐 Network

- Ensure ports **8081 / 8082** are open
- Configure proper SSL for WSS
- Set correct `ext-rtp-ip`

### 🎙️ Browser Support

- Requires WebRTC-enabled browsers
- HTTPS required for microphone access

## 🧠 Summary

| Component | Role |
|-----------|------|
| FreeSWITCH (mod_verto) | WebRTC signaling |
| WebSocket (WS/WSS) | Transport |
| verto-sdk.js | Browser client |
| Directory XML | User authentication |
| Dialplan | Call routing |

