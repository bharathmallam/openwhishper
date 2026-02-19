# Automator & Background Service Setup Guide

This guide shows how to run OpenWhispr automatically on system startup across different platforms.

## Option 1: macOS with Automator App (Easiest GUI)

### Create the Automator App

1. **Open Automator**
   - Press `Cmd + Space`, type "Automator", press Enter

2. **Create New Document**
   - Choose **Application** as the document type

3. **Add a "Run Shell Script" Action**
   - Search for "Run Shell Script" in the Actions library
   - Double-click to add it

4. **Paste the Script**
   ```bash
   export PATH=/usr/local/bin:/opt/homebrew/bin:$PATH
   cd "/path/to/openwhispr"
   npm run dev >> /tmp/openwhispr-dev.log 2>> /tmp/openwhispr-dev-error.log &
   ```
   
   Replace `/path/to/openwhispr` with your actual project path (e.g., `/Users/bharathmallam/Documents/Openwhisper/openwhispr`)

5. **Configure Options**
   - Change "Pass input" from **to stdin** to **as arguments** (if desired)
   - Enable "Show this action when the workflow runs" for visibility

6. **Save the App**
   - Press `Cmd + S`
   - Name: `OpenWhispr Dev Server`
   - File Format: **Application**
   - Save to: `~/Applications/`

### Add App to Login Items (Auto-start at Boot)

1. Open **System Settings → General → Login Items**
2. Click **+** under "Open at Login"
3. Navigate to `~/Applications/` and select `OpenWhispr Dev Server.app`
4. Restart your Mac to verify it launches automatically

### View Logs
```bash
tail -f /tmp/openwhispr-dev.log
tail -f /tmp/openwhispr-dev-error.log
```

---

## Option 2: macOS with LaunchAgent (Advanced - System Control)

Create `/Users/YOUR_USERNAME/Library/LaunchAgents/com.openwhispr.dev.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>com.openwhispr.dev</string>
	<key>ProgramArguments</key>
	<array>
		<string>/bin/bash</string>
		<string>-c</string>
		<string>cd /path/to/openwhispr && /usr/local/bin/npm run dev</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
	<key>KeepAlive</key>
	<true/>
	<key>StandardOutPath</key>
	<string>/tmp/openwhispr-dev.log</string>
	<key>StandardErrorPath</key>
	<string>/tmp/openwhispr-dev-error.log</string>
	<key>EnvironmentVariables</key>
	<dict>
		<key>PATH</key>
		<string>/usr/local/bin:/opt/homebrew/bin:/usr/bin:/bin</string>
	</dict>
</dict>
</plist>
```

Then load it:
```bash
launchctl load ~/Library/LaunchAgents/com.openwhispr.dev.plist
```

---

## Option 3: macOS with pm2 (Recommended - Cross-Platform)

See [SETUP.md](SETUP.md#running-as-a-background-service-macosx) for detailed pm2 instructions.

Quick setup:
```bash
npm install -g pm2
pm2 start "npm run dev" --name openwhispr --cwd /path/to/openwhispr
pm2 save
pm2 startup
# Follow the command it suggests (requires sudo)
```

---

## Option 4: Windows with Task Scheduler

### Create Batch File

Create `C:\Users\YOUR_USERNAME\openwhispr-start.bat`:

```batch
@echo off
cd C:\path\to\openwhispr
npm run dev >> C:\Temp\openwhispr-dev.log 2>&1
```

### Schedule with Task Scheduler

1. Open **Task Scheduler** (search in Start menu)
2. Click **Create Task**
3. **General tab:**
   - Name: `OpenWhispr Dev Server`
   - Check "Run whether user is logged in or not"
   - Check "Run with highest privileges"

4. **Triggers tab:**
   - New → At log on → Specific user (you) → OK

5. **Actions tab:**
   - New → Action: Start a program
   - Program: `C:\Users\YOUR_USERNAME\openwhispr-start.bat`
   - OK

6. Click **OK** to save

### View Logs
```cmd
type C:\Temp\openwhispr-dev.log
```

---

## Option 5: Linux with Systemd (Recommended)

Create `/etc/systemd/user/openwhispr.service`:

```ini
[Unit]
Description=OpenWhispr Dev Server
After=network.target

[Service]
Type=simple
User=%i
WorkingDirectory=/home/YOUR_USERNAME/openwhispr
ExecStart=/usr/bin/npm run dev
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=default.target
```

Then:
```bash
# Replace YOUR_USERNAME with your actual username
systemctl --user enable openwhispr.service
systemctl --user start openwhispr.service

# View status
systemctl --user status openwhispr.service

# View logs
journalctl --user -u openwhispr.service -f
```

---

## Option 6: Linux with Cron

Add to crontab:
```bash
crontab -e
```

Add this line:
```
@reboot cd /path/to/openwhispr && nohup npm run dev >> /tmp/openwhispr-dev.log 2>&1 &
```

---

## Option 7: Windows with pm2 (Cross-Platform)

Same as macOS:
```bash
npm install -g pm2
pm2 start "npm run dev" --name openwhispr --cwd C:\path\to\openwhispr
pm2 save
pm2 startup windows
```

---

## Comparison Table

| Platform | Method | Auto-restart | Ease | Cross-Platform |
|----------|--------|--------------|------|----------------|
| **macOS** | Automator (GUI) | ❌ | ⭐⭐⭐ | ❌ |
| **macOS** | LaunchAgent | ✅ | ⭐⭐ | ❌ |
| **macOS** | pm2 | ✅ | ⭐⭐ | ✅ |
| **Windows** | Task Scheduler | ✅ | ⭐⭐ | ❌ |
| **Windows** | pm2 | ✅ | ⭐⭐ | ✅ |
| **Linux** | systemd | ✅ | ⭐⭐ | ❌ |
| **Linux** | cron | ✅ | ⭐ | ❌ |
| **All OS** | pm2 | ✅ | ⭐⭐ | ✅ |

---

## Recommended Setup by Platform

### macOS (Easiest)
👉 **Use Automator App** (GUI, no terminal) or **pm2** (auto-restart on crash)

### Windows
👉 **Use Task Scheduler** (native) or **pm2** (better restart handling)

### Linux
👉 **Use systemd** (standard) or **pm2** (cross-platform)

### All Platforms (Best for Teams)
👉 **Use pm2** (same commands everywhere, better logging, easy management)

---

## Troubleshooting

### App doesn't start
- Check log files: `/tmp/openwhispr-dev.log`
- Verify the path to your project is correct
- Ensure `npm` is in the PATH

### Permission denied errors
- macOS: Grant Full Disk Access in System Settings
- Windows: Run Task Scheduler as Administrator
- Linux: Check file permissions: `chmod +x ~/openwhispr-start.sh`

### Port already in use
- Default port: 5183
- Kill existing process: `lsof -i :5183` then `kill -9 <PID>`
- Or change port in Vite config

### Logs not writing
- Check directory permissions: `ls -la /tmp/`
- Ensure log paths are writable

---

See [SETUP.md](SETUP.md) for installation and general troubleshooting.
