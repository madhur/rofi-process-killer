# Rofi Process Killer Module Setup

## Installation

1. **Save the script** to a location in your PATH:
   ```bash
   # Make directory for custom scripts (if it doesn't exist)
   mkdir -p ~/.local/bin
   
   # Save the script (copy from the artifact above)
   nano ~/.local/bin/rofi-process-killer.sh
   
   # Make it executable
   chmod +x ~/.local/bin/rofi-process-killer.sh
   ```

2. **Ensure ~/.local/bin is in your PATH** (add to ~/.bashrc or ~/.zshrc if needed):
   ```bash
   export PATH="$HOME/.local/bin:$PATH"
   ```

## Usage

### Basic Usage
Run the process killer with rofi:
```bash
rofi -show-icons -modi "processes:rofi-process-killer.sh" -show processes
```

### Enhanced Usage with Full Command Line Display
Save the custom theme (from the theme artifact) to `~/.config/rofi/process-killer.rasi`, then run:
```bash
rofi -modi "processes:rofi-process-killer.sh" \
     -show processes \
     -theme ~/.config/rofi/process-killer.rasi
```

### Testing Text Wrapping
To verify that wrapping is working, look for processes with very long command lines (like browsers, IDEs, or Java applications). You can also test with this command that forces wrapping:
```bash
# Force element text wrapping with inline styling
rofi -modi "processes:rofi-process-killer.sh" \
     -show processes \
     -theme-str 'window {width: 60%;} element-text {wrap: true; font: "monospace 9";} listview {dynamic: true; fixed-height: false;}'
```

### Create a Keyboard Shortcut
Add this to your window manager config (i3, sway, etc.) or use your DE's keyboard shortcuts:
```bash
# For i3/sway config (with custom theme)
bindsym $mod+Shift+p exec rofi -modi "processes:rofi-process-killer.sh" -show processes -theme ~/.config/rofi/process-killer.rasi

# For general use without custom theme
rofi -modi "processes:rofi-process-killer.sh" -show processes -theme-str 'element-text {wrap: true;} window {width: 95%;}'
```

## Features

- **Full Command Lines**: Shows complete command lines with all arguments (no truncation)
- **Text Wrapping**: Long command lines wrap to multiple lines for full visibility
- **Process List**: Shows PID, CPU%, Memory%, User, and full Command
- **Smart Sorting**: Processes sorted by CPU usage (highest first)
- **Safe Killing**: First attempts graceful termination (SIGTERM), then force kill (SIGKILL) if needed
- **Notifications**: Desktop notifications for kill status
- **Kernel Thread Filtering**: Excludes kernel threads from the list
- **Error Handling**: Validates PIDs and provides error feedback
- **Custom Theme**: Optimized for readability with wrapped text

## Process List Format
```
PID | CPU% | MEM% | USER | FULL_COMMAND_WITH_ALL_ARGUMENTS
1234 | 15.2% | 8.3% | username | /usr/bin/firefox --new-instance --profile /home/user/.mozilla/firefox/default --no-remote
5678 | 5.1% | 12.1% | username | /usr/bin/code --extensions-dir /home/user/.vscode/extensions --user-data-dir /home/user/.config/Code
```

Note: Long command lines will wrap to multiple lines in the rofi interface for full visibility.

## Customization Options

### Modify Process Selection
Edit the `ps aux` command in `get_processes()` function:
```bash
# Show only processes for current user
ps ux --no-headers | awk '...'

# Show processes with different formatting
ps aux --no-headers -o pid,pcpu,pmem,user,comm,args | awk '...'
```

### Change Sorting
Modify the sort command:
```bash
# Sort by memory usage
sort -k4 -nr

# Sort by PID
sort -k1 -n

# Sort alphabetically by command
sort -k5
```

### Custom Rofi Theme
Create a custom theme file `~/.config/rofi/process-killer.rasi`:
```css
* {
    background-color: #1e1e1e;
    text-color: #ffffff;
    border-color: #444444;
}

window {
    width: 90%;
    padding: 20px;
}

listview {
    lines: 20;
    scrollbar: true;
}

element selected {
    background-color: #ff5555;
    text-color: #ffffff;
}
```

Then use it:
```bash
rofi -modi "processes:rofi-process-killer.sh" -show processes -theme ~/.config/rofi/process-killer.rasi
```

## Dependencies

- `rofi` - The application launcher
- `ps` - Process listing (standard on all Linux systems)
- `awk` - Text processing (standard on all Linux systems)
- `notify-send` - Desktop notifications (usually pre-installed)

## Troubleshooting

1. **Permission Denied**: Some system processes require root privileges to kill
2. **Process Not Found**: Process may have already exited between listing and selection
3. **Notifications Not Working**: Install `libnotify-bin` package if missing

## Security Note

This script can kill any process the user has permission to kill. Use carefully and avoid killing critical system processes.