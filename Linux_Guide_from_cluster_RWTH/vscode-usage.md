# VS Code Usage

-------------------------------

## Remote server

### Remote server settings
- How to open it?
  
  After Login to remote server, Go to `File > Preferences > Settings` (or Ctrl+,).


- My current settings
    > Note that this setting is important for open zsh in remote terminal
    ```json
    {
        "editor.fontSize": 16,
        "python.terminal.activateEnvInCurrentTerminal": true,
        "python.terminal.activateEnvironment": false,
        "terminal.integrated.profiles.linux": {
                "zsh": {
                "path": "/usr/local_rwth/bin/zsh"
            }
        },
        "terminal.integrated.defaultProfile.linux": "zsh",
        "terminal.integrated.inheritEnv": false,
        "terminal.integrated.automationProfile.linux": {
            "path": "/usr/local_rwth/bin/zsh"
        },
        "terminal.integrated.splitCwd": "workspaceRoot"
    }
    ```
    There is also a way to set it globally in local `setting.json`
    ```json
    "[remote]": {
    "terminal.integrated.defaultProfile.linux": "bash"
    }
    ```



