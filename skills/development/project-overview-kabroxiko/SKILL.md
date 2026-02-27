---
name: project-overview
description: A protocol for creating a new IoT LED controller project from scratch, including setting up the folder structure, initializing PlatformIO for the firmware, and setting up a React/Vite project for the web UI.
license: MIT
metadata:
  author: GitHub Copilot
  version: "3.0"
---

## Agentic Project Scaffolding Protocol

As an AI assistant, your goal is to create a new, fully structured project for an IoT LED controller with a web-based UI. When the user asks to "create a new LED controller project" or similar, follow this protocol.

### Step 1: Gather Requirements from the User

1.  Ask the user for a project name. This will be used for the root directory.
2.  Confirm the target microcontroller. Default to `esp32dev` for an ESP32.
3.  Inform the user that you will create a two-part project: a C++ firmware backend and a React/Vite web frontend.

### Step 2: Create the Project Structure

Execute the following commands to create the necessary directories.

1.  **Create Root Directory:**
    ```bash
    mkdir <project-name>
    cd <project-name>
    ```
2.  **Create Subdirectories:**
    ```bash
    mkdir src
    mkdir web
    mkdir scripts
    ```

### Step 3: Initialize the Firmware Backend

1.  **Initialize PlatformIO:** Run this command in the project's root directory.
    ```bash
    platformio project init --board esp32dev
    ```
    This will create `platformio.ini` and other necessary files.
2.  **Create `main.cpp`:** Create a placeholder `src/main.cpp` file with a minimal Arduino sketch.
    ```cpp
    #include <Arduino.h>

    void setup() {
      Serial.begin(115200);
      Serial.println("Hello from the new project!");
    }

    void loop() {
      delay(1000);
    }
    ```

### Step 4: Initialize the Web UI Frontend

1.  **Initialize Vite Project:** Run this command from the project root to create a React project inside the `web` directory.
    ```bash
    npm create vite@latest web -- --template react
    ```
2.  **Install Dependencies:**
    ```bash
    npm install --prefix web
    ```

### Step 5: Create Helper Scripts

1.  **Create `embed_assets.py`:** Create a placeholder Python script in the `scripts/` directory. This script will eventually be used to embed the web UI into the firmware.
    ```python
    # scripts/embed_assets.py
    print("This script will be used to embed the web assets.")
    # TODO: Implement the logic to convert web/dist files into C++ headers.
    ```

### Step 6: Conclude and Inform the User

1.  Summarize the project structure you have created.
2.  Provide the user with the next steps:
    *   "You can now start developing the firmware in the `src` directory."
    *   "To work on the web UI, `cd web` and run `npm run dev`."
    *   "Remember to build the web UI and run the `embed_assets.py` script before deploying the firmware."
