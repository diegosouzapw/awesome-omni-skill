---
name: physic-claw-vea
description: An interactive 3D visualization application featuring an "Augmented Entity" with reactive shaders and emotional states.
---

# PhysicClaw-VEA

## Overview
PhysicClaw-VEA is an advanced interactive 3D experience built with **React Three Fiber** and **Three.js**. The core of the application is an "Augmented Entity" that dynamically reacts to simulated internal states such as "thinking", emotions, and intensity levels through custom shaders and animations.

## Key Features

### 🎨 Advanced 3D Visualization
- Immersive scene rendering using React Three Fiber.
- Support for loaded 3D models (GLB/FBX) with animation playback.
- Fallback to procedural geometry when models are loading or unavailable.

### ⚡ Reactive Shaders (`EnergyShader`)
- Custom GLSL shaders that visually modify the entity in real-time.
- Visual parameters include:
    - **Intensity**: Fluctuating energy levels.
    - **Thinking State**: Visual pulses indicating processing.
    - **Mood**: Color shifts based on the entity's emotional state.

### 🧠 "Soul" System (State Management)
- Powered by **Zustand** for global state management (`soulStore`).
- Simulates the entity's internal "soul" or consciousness, driving the visual changes and behaviors.

### 💬 Chat Interface
- An overlay interface allowing users to interact with the entity via text.
- Connects directly to the entity's state to trigger responses or visual changes.

## Tech Stack
- **Frontend**: React (v19), TypeScript, Vite
- **3D Graphics**: Three.js, React Three Fiber, Dre
- **State**: Zustand
- **Styling**: Tailwind CSS (configured)

## Installation & Usage

1.  **Install dependencies**:
    ```bash
    npm install
    ```

2.  **Run locally**:
    ```bash
    npm run dev
    ```

3.  **Build**:
    ```bash
    npm run build
    ```
