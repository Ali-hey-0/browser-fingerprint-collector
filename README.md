# Browser Data Collection Script

This is a comprehensive browser data collection script that gathers extensive information about the user's device, environment, and behavior.

## Features

### 📊 Data Collection Capabilities

**Device Information:**
- User agent, platform, and language details
- Screen resolution and color depth
- Device pixel ratio and hardware concurrency
- Device memory and touch support detection
- Timezone and orientation information

**Network & Location:**
- IP address and geolocation data
- Network connection type and speed
- Local IP discovery via WebRTC
- City, region, country, and ISP details

**Hardware & Sensors:**
- Battery status and charging state
- Accelerometer and gyroscope data
- Gamepad detection
- WebGL and audio capabilities
- GPU information

**Permissions & Access:**
- Camera, microphone, and location permissions
- Clipboard access
- Notification and USB device status
- MIDI devices and WebXR support

**Browser Environment:**
- Canvas fingerprinting
- Font detection
- WebAssembly support
- Storage (cookies, localStorage, sessionStorage)
- History length

**User Interaction:**
- Keyboard inputs
- Mouse movements
- Real-time interaction tracking

### 🔧 Technical Features

- **Dual Communication**: WebSocket + HTTP POST
- **Telegram Integration**: Optional Telegram bot notifications
- **Media Capture**: Screenshot and camera photo capture
- **Error Handling**: Comprehensive error tracking and reporting
- **Retry Logic**: Automatic retry for failed requests
- **Data Encryption**: XOR-based data obfuscation

## ⚠️ Important Notes

**This script is designed for:**
- Security research and penetration testing
- Authorized monitoring and analytics
- Educational purposes

**Legal and Ethical Considerations:**
- Only use on systems you own or have explicit permission to test
- Comply with local privacy laws and regulations
- Inform users about data collection when required
- Use responsibly and ethically

## Installation

1. Clone the repository:
```bash
git clone https://github.com/yourusername/browser-fingerprint-collector.git
```

2. Deploy the HTML file to your web server
3. Configure the endpoints in the script:
   - Update WebSocket URL (`_0x3c4d.w`)
   - Set API endpoint (`_0x1a2b`)
   - Configure Telegram bot (if using)

## Configuration

The script uses a configuration object with the following structure:

```javascript
const _0x3c4d = {
    w: "wss://your-websocket-endpoint",
    a: "https://your-api-endpoint",
    t: {e: false, b: "bot_token", c: "chat_id"},
    // ... other settings
};
```

## Data Points Collected

| Category | Data Collected |
|----------|----------------|
| **System** | OS, Platform, Languages, Screen, CPU, Memory |
| **Network** | IP, Geolocation, Connection Type, Local IPs |
| **Hardware** | Battery, Sensors, GPU, WebGL, Audio |
| **Permissions** | Camera, Mic, Location, Notifications, USB |
| **Browser** | User Agent, Storage, History, Canvas FP |
| **Media** | Screenshots, Camera Photos, Microphone |
| **Interaction** | Keystrokes, Mouse Movements, Timing |

## Privacy & Security

- Data is obfuscated before transmission
- Optional secure WebSocket connections
- Configurable data collection modules
- No permanent local storage

## Legal Disclaimer

This tool should only be used for legitimate purposes such as:
- Authorized security testing
- Legal monitoring applications
- Academic research
- Personal analytics on owned devices

Always ensure compliance with applicable laws and obtain proper consent before deployment.

## Contributing

Please ensure any contributions maintain ethical standards and include proper documentation about data collection practices.

## License

[Specify your license - consider MIT or GPL for open source]
