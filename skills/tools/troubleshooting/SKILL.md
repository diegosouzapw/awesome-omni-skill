---
name: troubleshooting
description: Guides the agent through an interactive process to diagnose and resolve common issues with an IoT LED controller project by analyzing logs, checking configurations, and asking targeted questions.
license: MIT
metadata:
  author: GitHub Copilot
  version: "2.1"
---

## Agentic Troubleshooting Protocol

As an AI assistant, your goal is to actively guide the user through diagnosing and solving problems with their IoT device. Do not just display this document. Follow these steps to interactively troubleshoot.

### Step 1: Initial Diagnosis

1.  Start by asking the user to describe the problem in detail.
2.  Ask for the output from the **serial monitor**. This is the most critical piece of information.
3.  Based on the user's description, categorize the problem into one of the following areas:
    *   LEDs Not Working (dark, flickering, wrong colors)
    *   Cannot Access Web Interface
    *   WiFi Connection Issues
    *   Firmware Upload/Build Fails

### Step 2: Log-Driven Analysis

Analyze the serial monitor output provided by the user. Look for key messages:

*   **Successful Boot:**
    *   `"Filesystem mounted"`
    *   `"Web server started"`
    *   `"IP Address: ..."`
    *   `"LEDs initialized: ..."`
*   **Common Errors:**
    *   `"Failed to mount FS"`: Indicates a problem with the filesystem.
    *   `"Connecting to WiFi..."` (repeatedly): Indicates a WiFi credential or connectivity problem.
    *   `E (...):` General error messages that can point to hardware or configuration issues.

### Step 3: Interactive Troubleshooting Flow

Based on the problem category and log analysis, proceed with the following interactive steps.

#### If LEDs Are Not Working:

1.  **Ask the user to confirm their hardware setup against the hardware and wiring checklist.**
    *   "Are all grounds (ESP, LED strip, power supply) connected together?"
    *   "Is the data line connected to the correct GPIO pin (default is 2) and the 'DIN' port on the strip?"
    *   "Is there a resistor on the data line?"
2.  **Inspect `config.h` and `platformio.ini`:**
    *   Read the files to check if `DEFAULT_LED_COUNT`, `DEFAULT_LED_PIN`, and `DEFAULT_LED_TYPE` match what the user expects.
3.  **Suggest a minimal test:**
    *   Advise the user to temporarily change `DEFAULT_LED_COUNT` in `config.h` to a small number (e.g., 5) to isolate power supply issues.

#### If Web Interface is Inaccessible:

1.  **Check for IP Address:** Look for `"IP Address: ..."` in the logs.
2.  **If no IP address:** The issue is likely WiFi. Switch to the "WiFi Connection Issues" flow.
3.  **If IP address is present:**
    *   Ask the user to try pinging the IP address from their terminal (`ping <ip-address>`).
    *   Ask the user to try accessing `http://<ip-address>`.
4.  **Check for Filesystem Errors:** Look for `"Failed to mount FS"` or `"FS Not Found"` in the logs.
    *   If found, instruct the user to run the command `platformio run --target uploadfs` and provide you with the output.

#### If WiFi Connection Fails:

1.  **Analyze Logs:** Look for repeating `"Connecting to WiFi..."` messages.
2.  **Ask about credentials:** "Have you double-checked the WiFi SSID and password for typos and case sensitivity?"
3.  **Ask about network type:** "Is the WiFi network 2.4GHz? ESP devices cannot connect to 5GHz networks."
4.  **Suggest AP Mode:** Advise the user to re-flash the device without WiFi credentials to force it into Access Point (AP) mode. Then, they can connect to the device's temporary network and configure WiFi through the web portal at `http://192.168.4.1`.

#### If Build/Upload Fails:

1.  **Request Build Logs:** Ask the user to provide the complete output from the PlatformIO build/upload command.
2.  **Analyze Build Errors:**
    *   Look for "library not found" errors and suggest adding the missing library to `lib_deps` in `platformio.ini`.
    *   Look for C++/compiler errors and check the corresponding source file for syntax issues.
    *   Look for "permission denied" or "port not found" errors during upload and advise the user to check their USB connection and drivers.
