---
name: rc-submarine-controller
description: Development skill for building a Raspberry Pi Pico-based RC submarine controller with ballast control, depth hold, pitch stabilization, and failsafe protection. Use when working on embedded C firmware for RP2040, implementing PID controllers, writing sensor drivers (pressure, IMU), PWM capture/output, dual-core safety-critical systems, or any task related to the RC submarine project. Also triggers for Power of 10 coding standards, Pico SDK usage, and underwater vehicle control systems.
---

# RC Submarine Controller Development Skill

Firmware development for a Raspberry Pi Pico-based RC submarine control system with active ballast, depth hold, pitch stabilization, and hardware failsafe protection.

## Project Overview

- **Platform**: Raspberry Pi Pico (RP2040 dual-core ARM Cortex-M0+)
- **Input**: Standard 6-channel RC receiver (PWM)
- **Sensors**: MS5837 pressure sensor, MPU-6050 IMU, leak detector, battery monitor
- **Actuators**: Ballast pump, solenoid valve, control surface servos
- **Architecture**: Dual-core (Core 0: safety monitor, Core 1: control logic)

## Quick Reference

### Pin Assignments

See `references/pinout.md` for complete GPIO mapping.

### Build Commands

```bash
# Configure
mkdir build && cd build
cmake -DPICO_SDK_PATH=~/pico-sdk ..

# Build
make -j4

# Flash (with Pico in BOOTSEL mode)
cp rc_sub_controller.uf2 /media/RPI-RP2

# Static analysis
cppcheck --enable=all --error-exitcode=1 src/
```

### Project Structure

```
rc-sub-controller/
├── CMakeLists.txt
├── src/
│   ├── main.c              # Entry, dual-core setup
│   ├── config.h            # Pin assignments, constants
│   ├── types.h             # Shared data types
│   ├── drivers/            # Hardware abstraction
│   ├── control/            # PID, state machines
│   ├── safety/             # Core 0 safety monitor
│   └── util/               # Logging, CRC, assertions
├── pio/                    # PIO programs
└── test/                   # Unit tests
```

## Coding Standards (Power of 10 Lite)

Apply these mandatory rules to all code:

1. **No dynamic memory after init** - Use static buffers only; no malloc/free in control loops
2. **Fixed loop bounds** - All loops must have provable iteration limits
3. **Check all return values** - Never ignore errors; propagate up call chain
4. **Compile with `-Wall -Werror`** - All warnings are errors
5. **Run cppcheck** - Zero errors before commit
6. **Assert preconditions** - Minimum 1 assert per function

### Assert Macro

```c
#define P10_ASSERT(cond) do { \
    if (!(cond)) { \
        log_fatal("ASSERT: %s:%d: %s", __FILE__, __LINE__, #cond); \
        trigger_emergency_blow(EVT_ASSERT_FAIL); \
    } \
} while(0)
```

Assert failures trigger emergency surface—never hang.

## Dual-Core Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  CORE 0 (Safety)              │  CORE 1 (Control)              │
│  • Watchdog feeding           │  • RC input capture            │
│  • Fault detection            │  • Sensor polling              │
│  • Emergency blow             │  • PID loops                   │
│  • Battery/leak monitor       │  • State machine               │
│  • 100 Hz loop                │  • 50 Hz loop                  │
│  • CANNOT be blocked          │  • Can be complex              │
└─────────────────────────────────────────────────────────────────┘
                    Shared State (lock-free, atomic flags)
```

Core 0 runs independently—if Core 1 crashes, Core 0 still triggers emergency surface.

## Driver Implementation Guide

### RC Input (PIO-based PWM Capture)

Use PIO for accurate PWM measurement. See `references/drivers.md` for complete implementation.

```c
// Output structure
typedef struct {
    uint16_t channels[6];  // 1000-2000 µs
    uint32_t timestamp_ms;
    bool valid;
} RcFrame_t;

// Convert to signed percent
int8_t pwm_to_percent(uint16_t pulse_us) {
    int16_t centered = (int16_t)pulse_us - 1500;
    int16_t scaled = (centered * 100) / 500;
    return (int8_t)CLAMP(scaled, -100, 100);
}
```

### Pressure Sensor (MS5837)

I2C driver with temperature compensation. See `references/drivers.md`.

```c
typedef struct {
    int32_t depth_cm;      // Depth in centimeters
    int32_t temp_c_x10;    // Temperature × 10
    uint32_t timestamp_ms;
    bool valid;
} DepthReading_t;
```

### IMU (MPU-6050)

Complementary filter for pitch/roll. See `references/drivers.md`.

```c
typedef struct {
    int16_t pitch_deg_x10;  // Pitch × 10 (0.1° resolution)
    int16_t roll_deg_x10;   // Roll × 10
    uint32_t timestamp_ms;
    bool valid;
} AttitudeReading_t;
```

## Control System

### PID Controller

Generic PID with anti-windup. See `references/control.md`.

```c
typedef struct {
    float kp, ki, kd;
    float integral;
    float prev_error;
    float integral_limit;
    float output_min, output_max;
} PidController_t;

float pid_update(PidController_t* pid, float setpoint, float measurement, float dt);
void pid_reset(PidController_t* pid);
```

### State Machine

See `references/control.md` for complete state machine specification.

```
States: INIT → SURFACE → DIVING → SUBMERGED_MANUAL
                                → SUBMERGED_DEPTH_HOLD
                          ↓
                     SURFACING → SURFACE
                          ↓
        Any state → EMERGENCY (on fault)
```

## Safety System

### Failsafe Triggers

Any of these conditions trigger emergency surface:

1. **Signal loss** - No valid RC frame for >3 seconds
2. **Low battery** - Voltage below threshold (e.g., 6.4V for 2S LiPo)
3. **Leak detected** - Water sensor triggers
4. **Depth exceeded** - Beyond configured max depth
5. **Pitch exceeded** - Beyond ±45° (configurable)

### Emergency Blow Sequence

```c
void trigger_emergency_blow(uint8_t reason) {
    // Atomic flag—cannot be undone except by power cycle
    g_emergency_active = true;
    
    // 1. Open vent valve (release air from ballast)
    valve_open();
    
    // 2. Run pump in reverse (expel water)
    pump_set_speed(-100);
    
    // 3. Full up on control surfaces
    servo_set_position(SERVO_BOWPLANE, +100);
    servo_set_position(SERVO_STERNPLANE, +100);
    
    // 4. Log the event
    log_event(EVT_EMERGENCY_BLOW, reason);
}
```

## Task Checklist

Execute `scripts/task_status.py` to see current progress. Tasks are independent—complete in any order.

### Foundation (F1-F5)
- [ ] F1: Project scaffold (CMakeLists.txt, directories)
- [ ] F2: Blink test (confirm toolchain)
- [ ] F3: Dual-core hello (both cores running)
- [ ] F4: Config header (pin assignments)
- [ ] F5: Types header (shared structs)

### Drivers (D1-D8)
- [ ] D1: RC input driver (PIO PWM capture)
- [ ] D2: Pressure sensor (MS5837 I2C)
- [ ] D3: IMU driver (MPU-6050 I2C)
- [ ] D4: Pump driver (PWM output)
- [ ] D5: Valve driver (GPIO)
- [ ] D6: Servo driver (PWM output)
- [ ] D7: Battery monitor (ADC)
- [ ] D8: Leak detector (GPIO interrupt)

### Control (C1-C5)
- [ ] C1: PID module
- [ ] C2: Depth controller
- [ ] C3: Pitch controller
- [ ] C4: Ballast state machine
- [ ] C5: Main state machine

### Safety (S1-S4)
- [ ] S1: Safety monitor (Core 0)
- [ ] S2: Emergency blow sequence
- [ ] S3: Watchdog setup
- [ ] S4: Fault logging

### Integration (I1-I5)
- [ ] I1: Wire everything
- [ ] I2: Bench test
- [ ] I3: Dry run
- [ ] I4: Wet test (bathtub)
- [ ] I5: Pond test

## File Generation

When generating code files for this project:

1. **Always include the standard header**:
   ```c
   /**
    * RC Submarine Controller
    * [filename] - [brief description]
    * 
    * Power of 10 compliant
    */
   ```

2. **Use the type definitions from types.h** - See `references/types.md`

3. **Follow the naming conventions**:
   - Functions: `module_action()` (e.g., `pump_set_speed()`)
   - Types: `CamelCase_t` (e.g., `RcFrame_t`)
   - Constants: `UPPER_CASE` (e.g., `MAX_DEPTH_CM`)
   - Files: `lowercase.c/.h`

4. **Run the validation script** after generating code:
   ```bash
   scripts/validate_code.py path/to/file.c
   ```

## Claude CLI Quick Start

If using Claude CLI (Claude Code), see `CLAUDE.md` for full instructions.

```bash
# One-time setup
./scripts/setup.sh              # Install all dependencies
python3 scripts/init_project.py . # Initialize project structure  
make install-hooks              # Setup git hooks

# Daily workflow
make ci-quick                   # Quick checks
make test                       # Run unit tests
make build                      # Build firmware
```

The CI system enforces Power of 10 rules automatically on commit.

## Reference Documents

- `CLAUDE.md` - Claude CLI onboarding guide
- `references/pinout.md` - Complete GPIO pin assignments
- `references/drivers.md` - Driver implementation details
- `references/control.md` - PID and state machine specifications
- `references/types.md` - All type definitions
- `references/safety.md` - Safety system specifications
- `references/hardware.md` - BOM and wiring diagrams

## CI System

- `ci/run_ci.sh` - Main CI runner (all checks)
- `ci/p10_check.py` - Power of 10 compliance checker
- `ci/install_hooks.sh` - Git hook installer
- `test/run_tests.sh` - Unit test runner
- `test/test_harness.h` - Minimal test framework

## Scripts

- `scripts/init_project.py` - Initialize project directory structure
- `scripts/validate_code.py` - Check Power of 10 compliance
- `scripts/task_status.py` - Show task completion status
- `scripts/generate_driver.py` - Generate driver boilerplate

## Related Documents

This skill supports development based on:
- RC_Submarine_Controller_Spec_NASA_JPL_v2.docx (full specification)
- RC_Submarine_Controller_Kickoff.docx (project plan)
