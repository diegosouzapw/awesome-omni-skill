---
name: serial-terminal
description: Safely interact with serial ports (send/receive data). Use when the user needs to communicate with serial devices like Arduino, ESP32, or other microcontrollers. CRITICAL - Never read /dev/tty* devices directly.
allowed-tools: Bash(python:*)
---

# Serial Terminal Skill

Safely interact with serial ports without directly reading /dev/tty* devices (which would crash Claude Code).

## CRITICAL SAFETY RULES

1. **NEVER** read from `/dev/tty*` devices directly using `cat`, `head`, `tail`, or any other command
2. **NEVER** use the Read tool on `/dev/tty*` files
3. **ALWAYS** use the Python wrapper script for ALL serial operations

## Device Identification

**IMPORTANT: Always use `/dev/serial/by-id/` paths when available!**

When multiple devices are connected, raw paths like `/dev/ttyACM0` can change between reboots or when devices are reconnected. The `/dev/serial/by-id/` directory provides stable, descriptive paths that include the device serial number.

Example `/dev/serial/by-id/` entries:
```
/dev/serial/by-id/usb-SEGGER_J-Link_001057754341-if00 -> ../../ttyACM2
/dev/serial/by-id/usb-SEGGER_J-Link_001057754341-if02 -> ../../ttyACM3
```

**When the user asks to connect to a device:**
1. First run `list` to show available devices with their serial numbers
2. Ask the user which device/serial number they want to use if multiple are present
3. Use the `/dev/serial/by-id/` path for the connection

## Available Commands

The script is located at `~/.claude/skills/serial-terminal/scripts/serial_terminal.py`

### List Available Ports

```bash
python3 ~/.claude/skills/serial-terminal/scripts/serial_terminal.py list
```

This will show devices from `/dev/serial/by-id/` with their serial numbers, making it easy to identify which device is which.

### Send Data

```bash
python3 ~/.claude/skills/serial-terminal/scripts/serial_terminal.py send <port> "<message>" [options]
```

Options:
- `--baud <rate>`: Baud rate (default: 115200)
- `--newline`: Append newline to message (useful for command-line interfaces)

Example:
```bash
python3 ~/.claude/skills/serial-terminal/scripts/serial_terminal.py send "/dev/serial/by-id/usb-SEGGER_J-Link_001057754341-if02" "AT" --newline --baud 9600
```

### Receive Data

```bash
python3 ~/.claude/skills/serial-terminal/scripts/serial_terminal.py receive <port> [options]
```

Options:
- `--baud <rate>`: Baud rate (default: 115200)
- `--timeout <secs>`: How long to wait for data (default: 2.0)
- `--bytes <n>`: Max bytes to read (default: 4096)
- `--hex`: Display data as hex dump

Example:
```bash
python3 ~/.claude/skills/serial-terminal/scripts/serial_terminal.py receive "/dev/serial/by-id/usb-SEGGER_J-Link_001057754341-if02" --timeout 5 --baud 115200
```

### Send and Receive (most common)

Send a command and wait for response in one operation:

```bash
python3 ~/.claude/skills/serial-terminal/scripts/serial_terminal.py sendrecv <port> "<message>" [options]
```

Example:
```bash
python3 ~/.claude/skills/serial-terminal/scripts/serial_terminal.py sendrecv "/dev/serial/by-id/usb-SEGGER_J-Link_001057754341-if02" "help" --newline --timeout 3
```

## Troubleshooting

### Permission denied
The user may need to add themselves to the `dialout` group:
```bash
sudo usermod -a -G dialout $USER
# Then logout and login again
```

Or use sudo for a quick test:
```bash
sudo python3 ~/.claude/skills/serial-terminal/scripts/serial_terminal.py list
```

### No response received
- Check baud rate matches the device
- Increase timeout with `--timeout`
- Verify the device is connected with `list`
- Try adding `--newline` if the device expects CR/LF

### Device busy
Another program may have the port open. Close any serial monitors or other tools.

## Script Details

The Python script uses only standard library modules:
- `os` - Low-level file operations
- `termios` - Serial port configuration
- `select` - Non-blocking I/O (safe timeout handling)
- `fcntl` - File descriptor control

It safely handles serial I/O by:
1. Using `select()` to wait for data with timeout (never blocks indefinitely)
2. Setting non-blocking mode to prevent hangs
3. Properly restoring terminal settings on exit
