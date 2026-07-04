<div dir="ltr">

# Headless Claude Code Automation

Automation system for turning a MacBook into a remote-accessible AI-powered workstation.

## Technical Guide

### 1. Prerequisites

Install the required tools:
```bash
brew install tmux hammerspoon
```

Install Claude Code and log in/register:
https://code.claude.com/docs/en/quickstart

### 2. Session Persistence with Tmux
To prevent your Claude session from closing when you exit the terminal or lose connection, we use tmux.
* Open your terminal
* Create a new session named claude-dev: ```tmux new-session -s claude-dev```
* Inside this session, launch the agent: ```claude remote-control```
* To "detach" from the session (leaving it running in the background), press Ctrl+B then D.
* To return to this session later, run: ```tmux attach -t claude-dev```

### 3. Automating with Hammerspoon
Hammerspoon uses Lua to watch system events. This ensures that when you arrive at your designated workspace (e.g., your home office Wi-Fi), the agent starts automatically.
* Install Hammerspoon from hammerspoon.org and move it to your Applications folder.
* Open Hammerspoon, click the icon in the menu bar, and select Open Config File.
* Paste the following logic into your ```init.lua``` file:

    ```Lua
        -- Function to reset and start the agent
        local function startClaude()
            -- Kill any stale processes to avoid "Port Busy" errors
            os.execute("killall claude") 
            -- Start a new detached tmux session running Claude
            os.execute("tmux new-session -d -s claude-dev 'claude       remote-control'")
        end

        -- Watcher for your Wi-Fi network
        local wifiWatcher = hs.wifi.watcher.new(function()
            local currentSSID = hs.wifi.currentNetwork()
            -- Replace 'YOUR_WIFI_NAME' with your trusted network SSID
            if currentSSID == "YOUR_WIFI_NAME" then
                startClaude()
            end
        end)

        wifiWatcher:start()
    ```


### 4. Stability and System Availability
To treat your Mac as a true server, it must remain awake even when the display is off.

#### Preventing Sleep:
* **Quick Fix**: Open your terminal and run ```caffeinate -i```. This command prevents the system from entering idle sleep while the terminal window is active.
* **Permanent Fix**: Use an application like KeepingYouAwake (a GUI wrapper for ```caffeinate```). Enable it when you plug in your MacBook to ensure the machine never sleeps while connected to power.

#### Workspace Trust:
* Claude Code requires explicit permission to interact with your local folders. Open your terminal, navigate to your target project folder, and run ```claude``` once manually.
* Type ```y``` when prompted to "Trust" the workspace. This creates a configuration entry that allows the automated script to bypass manual approval prompts in the future.

#### Contextual Configuration:
* Create a ```CLAUDE.md``` file in the root of your project directory. This file acts as a "system prompt" for the agent, ensuring it understands your specific architectural patterns, tech stack, and coding style without needing repetitive instructions every time the session restarts.

</div>
