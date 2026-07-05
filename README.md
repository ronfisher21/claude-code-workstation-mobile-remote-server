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
* Inside this session, launch the agent (make sure you the following command NOT in "/home/your-user" or on "~/"):
   ```claude remote-control```
* To "detach" from the session (leaving it running in the background), press Ctrl+B then D.
* To return to this session later, run: ```tmux attach -t claude-dev```

### 3. Automating with Hammerspoon
Hammerspoon uses Lua to watch system events. This ensures that when you arrive at your designated workspace (e.g., your home office Wi-Fi), the agent starts automatically.
* Open Hammerspoon, click the icon in the menu bar, and select Open Config File.
* If the file doesnt exists please run the following command:
  ```bash
   mkdir -p ~/.config/nvim && vim ~/.config/nvim/init.lua
  ```
  press i (INSERT)
* Paste the following logic into your ```init.lua``` file:

    ```Lua
         -- Trusted networks: modify and/or add your wifi network
         local trustedNetworks = {
             ["MY-HOME-WIFI"] = true,
             -- ["MY_WORK-WIFI"] = true,
             -- ["HOME_BACKUP"] = true,
         }
         
         hs.location.start()
         
         local isStarting = false
         
         local function startClaude()
             if isStarting then return end
             isStarting = true
             os.execute("killall claude")
             os.execute("tmux kill-session -t claude-dev 2>/dev/null; sleep 0.5")
         
             -- check with "which claude" and "which tmux" your claude and tmux directories and put them below in the directories before tmux/claude commands
             os.execute("/opt/homebrew/bin/tmux new-session -d -s claude-dev '/opt/homebrew/bin/claude remote-control' >> /tmp/claude-hammerspoon.log 2>&1")
             hs.alert.show("✅ Claude remote-control session started")
         
             isStarting = false
         end
         
         
         local function checkConditions()
             local currentSSID = hs.wifi.currentNetwork()
             local onTrustedWifi = currentSSID ~= nil and trustedNetworks[currentSSID] == true
             local onACPower = hs.battery.powerSource() == "AC Power"
         
             if onTrustedWifi and onACPower then
                 startClaude()
             end
         end
         
         local wifiWatcher = hs.wifi.watcher.new(checkConditions)
         wifiWatcher:start()
         
         local powerWatcher = hs.battery.watcher.new(checkConditions)
         powerWatcher:start()
         
         checkConditions()
    ```


### 4. Stability and System Availability
To treat your Mac as a true server, it must remain awake even when the display is off.

#### Preventing Sleep:
* Use an application like KeepingYouAwake (a GUI wrapper for ```caffeinate```). Enable it when you plug in your MacBook to ensure the machine never sleeps while connected to power.
You can download it in the following link:
https://keepingyouawake.app
* Select the cup icon and enter settings.
* Go to Advanced and tick the "Allow the display to sleep" box.
* Go to macOs system setting:
  * battery => options
  * toggle ON "Prevent automatic sleeping on power adapter when the display if off."


#### Workspace Trust:
* Claude Code requires explicit permission to interact with your local folders. Open your terminal, navigate to your target project folder, and run ```claude``` once manually.
* Type ```y``` when prompted to "Trust" the workspace. This creates a configuration entry that allows the automated script to bypass manual approval prompts in the future.

#### Contextual Configuration:
* Create a ```CLAUDE.md``` file in the root of your project directory. This file acts as a "system prompt" for the agent, ensuring it understands your specific architectural patterns, tech stack, and coding style without needing repetitive instructions every time the session restarts.

</div>
