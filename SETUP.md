# OpenWhispr Setup & Installation Guide

## Prerequisites

Ensure you have the following installed on your system:

### macOS
```bash
# Install Homebrew (if not already installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Node.js (v18+) and npm
brew install node

# Install FFmpeg (required for audio conversion)
brew install ffmpeg
```

### Windows
1. Download and install [Node.js](https://nodejs.org/) (v18+)
2. Download and install [FFmpeg](https://ffmpeg.org/download.html) and add to PATH
3. Visual C++ Build Tools (for building native modules)

### Linux (Ubuntu/Debian)
```bash
# Install Node.js and npm
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install FFmpeg
sudo apt-get install -y ffmpeg

# Build tools
sudo apt-get install -y build-essential python3
```

## Installation Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/OpenWhispr/openwhispr.git
   cd openwhispr
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```
   
   If you encounter native module build errors on Mac, try:
   ```bash
   npm install --build-from-source
   ```

3. **Download Whisper models**
   ```bash
   npm run download:whisper-cpp
   ```
   
   This downloads the `whisper-server` binary for your platform and any required Whisper models.
   
   For all model architectures (optional):
   ```bash
   npm run download:whisper-cpp -- --all
   npm run download:llama-server -- --all
   npm run download:sherpa-onnx -- --all
   ```

4. **Configure environment variables (optional)**
   ```bash
   cp .env.example .env
   # Edit .env with your API keys (OpenAI, Anthropic, Google, Groq, etc.)
   ```

5. **Verify installation**
   ```bash
   # Check if whisper-server binary is available
   ls -la resources/bin/whisper-server-*
   
   # Verify FFmpeg is installed
   which ffmpeg
   ```

## Running the Application

### Development Mode (with hot reload)
```bash
npm run dev
```
The Electron app opens automatically. Vite dev server runs at http://127.0.0.1:5183

### Running as a Background Service (macOS/Linux with pm2)

**First-time setup:**
```bash
# Install pm2 globally
npm install -g pm2

# Create wrapper script
cat > ~/openwhispr-start.sh << 'EOF'
#!/bin/bash
export PATH=$PATH:/usr/local/bin:/opt/homebrew/bin
cd /path/to/openwhispr
npm run dev >> /tmp/openwhispr-dev.log 2>> /tmp/openwhispr-dev-error.log
EOF

chmod +x ~/openwhispr-start.sh

# Start with pm2
pm2 start ~/openwhispr-start.sh --name openwhispr --interpreter bash

# Auto-restart on reboot
pm2 save
pm2 startup
# Copy and run the command it suggests (requires sudo)
```

**Manage the service:**
```bash
pm2 status              # Check process status
pm2 logs openwhispr     # View logs
pm2 stop openwhispr     # Stop the process
pm2 start openwhispr    # Start the process
pm2 restart openwhispr  # Restart
```

### Production Build
```bash
npm run build
```
Builds the Electron app and renderer. Output in `dist/`.

## Permissions Required (macOS)

The app needs the following permissions:
- **Microphone**: For audio input
- **Accessibility**: For automatic text pasting
- **Input Monitoring**: For keyboard shortcuts

Grant these in **System Settings → Privacy & Security**:
1. Go to **Microphone** → add the app
2. Go to **Accessibility** → add the app
3. Go to **Input Monitoring** → add the app

## Environment Variables

Copy `.env.example` to `.env` and fill in your API keys:

```bash
# OpenAI API Key
OPENAI_API_KEY=sk-...

# Anthropic API Key (Claude)
ANTHROPIC_API_KEY=sk-ant-...

# Google Gemini API Key
GOOGLE_API_KEY=...

# Groq API Key
GROQ_API_KEY=gsk_...

# Optional: Debug mode
DEBUG=false
```

## Troubleshooting

### "whisper-server binary not found"
```bash
npm run download:whisper-cpp -- --current --force
```

### "FFmpeg is missing"
- **macOS**: `brew install ffmpeg`
- **Windows**: Download from https://ffmpeg.org and add to PATH
- **Linux**: `sudo apt-get install ffmpeg`

### "Cannot find module 'fdir'"
```bash
npm install fdir
```

### Permission errors on startup (LaunchAgent/pm2)
Ensure the script has execute permissions:
```bash
chmod +x ~/openwhispr-start.sh
```

### Accessibility permission denied on macOS
1. Open **System Settings → Privacy & Security → Accessibility**
2. Unlock the padlock
3. Click `+` and add the Electron app from `node_modules/electron/dist/Electron.app`
4. Restart the app

## Project Structure

```
openwhispr/
├── src/                     # React frontend (TypeScript/JSX)
│   ├── components/          # UI components
│   ├── helpers/             # Business logic (IPC handlers, transcription)
│   ├── hooks/               # React hooks
│   ├── config/              # Configuration and constants
│   └── locales/             # i18n translations
├── resources/               # Native binaries
│   └── bin/                 # whisper-server, ffmpeg, etc.
├── scripts/                 # Build and download scripts
├── main.js                  # Electron main process
├── preload.js               # Electron preload script
├── package.json             # Dependencies and scripts
└── README.md                # Project documentation
```

## Available npm Scripts

```bash
npm run dev                    # Start dev server with hot reload
npm run build                  # Build for production
npm run start                  # Run compiled app
npm run download:whisper-cpp   # Download whisper-server binary
npm run download:llama-server  # Download llama-cpp-server
npm run download:sherpa-onnx   # Download Sherpa-ONNX binary
npm run format                 # Format and lint code
npm run typecheck              # TypeScript type checking
npm run clean                  # Clean temporary files and models
```

## Support & Documentation

- **GitHub**: https://github.com/OpenWhispr/openwhispr
- **Issues**: https://github.com/OpenWhispr/openwhispr/issues
- **Docs**: See README.md for more details

---

**Last Updated**: February 2026
