---
name: jitx-pin-assignment
description: Model flexible pin assignment in JITX designs. Use when user asks about provide/require patterns, @provide.one_of or @provide.subset_of decorators, programmatic Provide, pin muxing (MCU peripherals on shared pins), DiffPair P/N polarity swapping, PCIe lane swapping or width variants, DDR4 byte/bit swapping, LPDDR5 channel swapping, hierarchical provider composition, topology (>>) on pin-assigned ports, ConstrainDiffPair or ConstrainReferenceDifference with provide/require, or flexible pin mapping for FPGAs and MCUs. Covers provide, Provide, require, all_of, one_of, subset_of, and protocol-specific pin flexibility with SI constraints.
---

# JITX Pin Assignment

Model flexible pin mappings between circuit-level interfaces and component-level pins. JITX's provide/require system lets the layout engine choose optimal pin assignments during routing.

## Environment

Environment setup is handled by the base `jitx` skill. Ensure it has been invoked first.

## Package Architecture

```python
# Pin assignment imports
from jitx.net import Port, DiffPair, provide, Provide

# Common bundles
from jitx.common import Power, GPIO

# Protocol bundles (for peripheral muxing)
from jitxlib.protocols.serial import I2C, SPI, UART

# SI constraint imports (for topology + constraint composition)
from jitx.si import (
    Constrain,
    ConstrainDiffPair,
    ConstrainReferenceDifference,
    DiffPairConstraint,
    ReferencePlanes,
)
from jitx.net import Topology

# Core framework
from jitx import Circuit, Net
```

**These DO NOT EXIST — never import:**
`jitx.provide`, `jitx.providers`, `jitx.pin_assignment`, `jitx.assign`,
`jitx.net.provide_one_of`, `jitxlib.pin_assignment`, `jitx.pin_assign`

**Key locations:**
- `provide` (lowercase, decorator) is in `jitx.net`
- `Provide` (uppercase, constructor class) is in `jitx.net` (also re-exported from `jitx`)
- `GPIO` is in `jitx.common`
- `I2C`, `SPI`, `UART` are in `jitxlib.protocols.serial`
- All constraint classes are in `jitx.si`

## When to Use Pin Assignment

Use pin assignment when a component's physical pins can validly serve more than one logical role, and you want the layout engine to choose the optimal mapping.

**Use pin assignment for:**
- MCU GPIO — any of N pins can drive an LED, sensor, etc.
- Peripheral muxing — I2C/SPI/UART available on alternate pin groups
- DiffPair polarity — some protocols allow P/N swap (PCIe, USB3)
- Lane ordering — PCIe lane reversal, width variants (x1/x2/x4)
- Byte/bit swapping — DDR controllers that support data bus reordering

**Use fixed wiring instead when:**
- The mapping is 1:1 with no flexibility (e.g., DDR address pins, CK polarity)
- The component datasheet specifies a single valid pin function
- You want a deterministic connection regardless of layout

## Bundles: The Language of Provide/Require

A **bundle** is a Port subclass that groups related signals. Bundles are the type parameter for `@provide` and `require()` — they define what interface is being offered and consumed.

### Built-in Bundles and Their Sub-Ports

Discover sub-ports by reading the class source: `grep -A 10 "class BundleName" .venv/lib/python*/site-packages/jitx*/`

| Bundle | Import | Sub-ports | Notes |
|--------|--------|-----------|-------|
| `GPIO` | `jitx.common` | `.gpio` | Single pin |
| `Power` | `jitx.common` | `.Vp`, `.Vn` | Power/ground pair |
| `DiffPair` | `jitx.net` | `.p`, `.n` | Positive/negative pair |
| `I2C` | `jitxlib.protocols.serial` | `.sda`, `.scl` | Always present |
| `SPI` | `jitxlib.protocols.serial` | `.sck`, `.copi`, `.cipo`, `.cs` | `cs`, `copi`, `cipo` are optional: `SPI(cs=True)` |
| `UART` | `jitxlib.protocols.serial` | `.tx`, `.rx`, `.cts`, `.rts`, ... | `tx`/`rx` default on; flow control optional: `UART(cts=True, rts=True)` |

### Defining Custom Bundles

When no built-in bundle matches your interface, subclass `Port`:

```python
from jitx.net import Port, DiffPair

class TXLink(Port):
    """Single TX differential pair."""
    tx = DiffPair()

class PCIeLane(Port):
    """Single PCIe lane: TX + RX diff pairs."""
    TX = DiffPair()
    RX = DiffPair()

class PCIeLink(Port):
    """Multi-lane PCIe link."""
    lane = [PCIeLane() for _ in range(4)]

class DDR4ByteLane(Port):
    """One DDR4 byte lane: DQ bits + strobe + mask."""
    DQ = [Port() for _ in range(8)]
    DQS = DiffPair()
    DM = Port()
```

**Rules for custom bundles:**
- Subclass `Port`, not `Circuit` or `Component`
- Sub-ports are class attributes (Port, DiffPair, or lists of them)
- Use lists for indexed groups: `DQ = [Port() for _ in range(8)]`
- Nest bundles for hierarchical interfaces: `lane = [PCIeLane() for _ in range(4)]`

## The Provide/Require Architecture

Pin assignment uses a three-tier architecture:

```
Component            Circuit Wrapper          Application Circuit
┌──────────┐        ┌──────────────────┐      ┌───────────────────┐
│ GPIO[0]  │←──────→│ @provide(GPIO)   │      │                   │
│ GPIO[1]  │   maps │  maps bundle     │◄─────│ .require(GPIO)    │
│ GPIO[2]  │  pins  │  ports to pins   │      │  gets a bundle    │
│ GPIO[3]  │        │                  │      │  instance to wire │
└──────────┘        └──────────────────┘      └───────────────────┘
 (physical)          (declares flexibility)    (consumes interface)
```

1. **Component** — defines physical pins (`Port()` class attributes)
2. **Circuit wrapper** — wraps the component, declares what interfaces it can provide and which pins map to each
3. **Application circuit** — calls `.require()` to acquire an interface, then wires its sub-ports

### The Mapping Return Type

Every `@provide` method returns a list of dictionaries. Each dictionary maps **bundle sub-ports → component pins**:

```python
@provide(GPIO)
def provide_gpio(self, g: GPIO):
    # g is a GPIO bundle instance — it has a .gpio sub-port
    # Each dict maps {bundle_sub_port: component_pin}
    return [
        {g.gpio: self.mcu.GPIO[0]},   # Option 1: GPIO[0] fulfills this GPIO
        {g.gpio: self.mcu.GPIO[1]},   # Option 2: GPIO[1] fulfills this GPIO
    ]

@provide.one_of(I2C)
def provide_i2c(self, i2c: I2C):
    # i2c has .sda and .scl — map BOTH in each option
    return [
        {i2c.sda: self.mcu.GPIO[0], i2c.scl: self.mcu.GPIO[1]},  # Option A
        {i2c.sda: self.mcu.GPIO[2], i2c.scl: self.mcu.GPIO[3]},  # Option B
    ]
```

**The keys** are always sub-ports of the bundle parameter (`g.gpio`, `i2c.sda`, etc.).
**The values** are always component pins from `self.<component>.<port>`.

## The Two APIs

### Decorator API (preferred for static configurations)

```python
from jitx.net import provide

class MCUCircuit(Circuit):
    @provide(GPIO)
    def provide_gpio(self, g: GPIO):
        return [{g.gpio: pin} for pin in self.mcu.GPIO]
```

### Constructor API (preferred for dynamic/computed port counts)

```python
from jitx.net import Provide

class MCUCircuit(Circuit):
    def __init__(self, num_gpio: int = 8):
        self.mcu = MCU()
        gpio_pins = self.mcu.GPIO[:num_gpio]
        self.gpios = Provide(GPIO).all_of(
            lambda g: [{g.gpio: pin} for pin in gpio_pins]
        )
```

**When to use which:**
- **Decorator**: Port count is known at class definition time
- **Constructor**: Port count depends on `__init__` parameters or runtime values

## Provider Patterns

### `@provide(Bundle)` — Multiple independent offers (all_of)

Creates one provider per mapping. Each can be independently assigned. Use for GPIO, where any pin can independently serve as a GPIO.

```python
@provide(GPIO)
def provide_gpio(self, g: GPIO):
    return [{g.gpio: pin} for pin in self.mcu.GPIO]
```

### `@provide.one_of(Bundle)` — Single selection from alternatives

Only ONE option from the returned list is selected. Use for peripheral pin muxing where an entire bus can appear on one pin group OR another, but not both.

```python
@provide.one_of(I2C)
def provide_i2c(self, i2c: I2C):
    return [
        {i2c.sda: self.mcu.GPIO[0], i2c.scl: self.mcu.GPIO[1]},  # Option A
        {i2c.sda: self.mcu.GPIO[2], i2c.scl: self.mcu.GPIO[3]},  # Option B
    ]
```

### `@provide.subset_of(Bundle, count)` — N from M

From the full set of mappings, at most `count` may be assigned. Use for resource-constrained scenarios (e.g., limited current budget across GPIO).

```python
@provide.subset_of(GPIO, 4)
def provide_gpio(self, g: GPIO):
    """8 GPIO pins available, but only 4 may be assigned."""
    return [{g.gpio: pin} for pin in self.mcu.GPIO]
```

### Constructor equivalents

```python
# all_of (same as @provide)
self.gpios = Provide(GPIO).all_of(lambda g: [{g.gpio: p} for p in pins])

# one_of (same as @provide.one_of)
self.i2c = Provide(I2C).one_of(lambda i2c: [
    {i2c.sda: self.mcu.GPIO[0], i2c.scl: self.mcu.GPIO[1]},
    {i2c.sda: self.mcu.GPIO[2], i2c.scl: self.mcu.GPIO[3]},
])

# subset_of (same as @provide.subset_of)
self.gpios = Provide(GPIO).subset_of(4, lambda g: [{g.gpio: p} for p in pins])
```

## Consuming Providers with `require()`

The consumer circuit calls `.require(BundleType)` on the provider circuit instance. This returns a bundle instance whose sub-ports can be wired with `+` or `>>`.

```python
class AppCircuit(Circuit):
    def __init__(self):
        self.mcu_circuit = MCUCircuit()

        # Request a GPIO — returns a GPIO bundle instance
        gpio = self.mcu_circuit.require(GPIO)

        # Wire using the bundle's sub-ports with + (net) operator
        self.led_net = gpio.gpio + self.led.anode

        # Request an I2C — returns an I2C bundle instance
        i2c = self.mcu_circuit.require(I2C)

        # Wire both sub-ports
        self.sda_net = i2c.sda + self.sensor.SDA
        self.scl_net = i2c.scl + self.sensor.SCL
```

## Complete End-to-End Example

```python
"""MCU with GPIO pin assignment driving 2 LEDs."""

from jitx import Circuit, Net
from jitx.common import GPIO, Power
from jitx.net import Port, provide
from jitxlib.parts import Resistor, Capacitor


class MCUComponent(jitx.Component):
    """8-pin MCU — physical component with pins."""
    VCC = Port()
    GND = Port()
    GPIO = [Port() for _ in range(4)]
    RESET = Port()
    NC = Port()
    # ... landpattern, symbol, mapping ...


class MCUCircuit(Circuit):
    """Wrapper that declares pin flexibility via @provide."""

    power = Power()

    @provide(GPIO)
    def provide_gpio(self, g: GPIO):
        """Any of 4 GPIO pins can independently serve as GPIO."""
        return [{g.gpio: pin} for pin in self.mcu.GPIO]

    def __init__(self):
        self.mcu = MCUComponent()
        self.VCC = Net(name="VCC")
        self.GND = Net(name="GND")
        self.VCC += self.power.Vp + self.mcu.VCC
        self.GND += self.power.Vn + self.mcu.GND

        self.c_bypass = Capacitor(capacitance=100e-9)
        self.c_bypass.insert(self.mcu.VCC, self.mcu.GND)

        self.r_reset = Resistor(resistance=10e3)
        self.r_reset.insert(self.mcu.VCC, self.mcu.RESET)


class LEDDriverApp(Circuit):
    """Application that consumes GPIOs to drive LEDs."""

    vin = Power()

    def __init__(self):
        self.GND = Net(name="GND")
        self.VCC = Net(name="VCC")
        self.GND += self.vin.Vn
        self.VCC += self.vin.Vp

        # Instantiate the provider circuit
        self.mcu = MCUCircuit()
        self.VCC += self.mcu.power.Vp
        self.GND += self.mcu.power.Vn

        # Consume GPIOs — layout engine decides which physical pins
        for i in range(2):
            gpio = self.mcu.require(GPIO)
            r = Resistor(resistance=330.0)
            setattr(self, f"r_led{i}", r)
            r.insert(gpio.gpio, ...)  # wire to LED anode

Device = LEDDriverApp
```

## Hierarchical Provider Composition

Providers that internally require from sub-providers. Use for peripheral muxing where signals (SDA, SCL) can be independently selected from different pin options.

```python
class MCUCircuit(Circuit):
    power = Power()

    # Step 1: Define inner bundle types for each muxed signal
    class I2C_SDA(Port):
        p = Port()

    class I2C_SCL(Port):
        p = Port()

    # Step 2: Provide each signal independently with one_of
    @provide.one_of(I2C_SDA)
    def provide_sda(self, sda: I2C_SDA):
        return [{sda.p: self.mcu.GPIO[0]}, {sda.p: self.mcu.GPIO[2]}]

    @provide.one_of(I2C_SCL)
    def provide_scl(self, scl: I2C_SCL):
        return [{scl.p: self.mcu.GPIO[1]}, {scl.p: self.mcu.GPIO[3]}]

    # Step 3: Compose into the real I2C bundle
    @provide(I2C)
    def provide_i2c(self, i2c: I2C):
        sda = self.require(self.I2C_SDA)  # self.require() consumes own providers
        scl = self.require(self.I2C_SCL)
        return [{i2c.sda: sda.p, i2c.scl: scl.p}]

    def __init__(self):
        self.mcu = MCUComponent()
        # ... power wiring ...
```

This creates 4 possible I2C configurations (2 SDA options x 2 SCL options) that the layout engine evaluates simultaneously.

**Key rules:**
- Inner Port classes are defined as nested classes on the Circuit
- `self.require()` inside `@provide` consumes the circuit's own providers
- The layout engine resolves the full constraint tree simultaneously

## DiffPair P/N Polarity Swapping

**Only model P/N swap when the part or protocol explicitly supports it.** See [references/protocol-pin-flexibility.md](references/protocol-pin-flexibility.md) for per-protocol rules.

When the component has individual P/N pins (not a DiffPair bundle), the Provide mapping can offer both polarities:

```python
class TXLink(Port):
    tx = DiffPair()

class FlexTXCircuit(Circuit):
    @provide.one_of(TXLink)
    def provide_tx(self, link: TXLink):
        return [
            {link.tx.p: self.ic.TXP, link.tx.n: self.ic.TXN},  # Normal
            {link.tx.p: self.ic.TXN, link.tx.n: self.ic.TXP},  # Swapped
        ]
```

## Topology and Constraints on Pin-Assigned Ports

Pin assignment and SI constraints compose naturally. The pattern is:
1. `require()` to get ports from a provider
2. `>>` to build topology on those ports
3. `Constrain` / `ConstrainDiffPair` / `ConstrainReferenceDifference` to apply SI constraints

### Basic pattern: DiffPair provide + topology + constraint

```python
class App(Circuit):
    def __init__(self):
        self.src = FlexTXCircuit()    # has @provide.one_of(TXLink)
        self.dst = DiffPairReceiver()

        tx = self.src.require(TXLink)

        # Build topology with >>
        self += tx.tx.p >> self.dst.INP.p
        self += tx.tx.n >> self.dst.INP.n

        # Constrain — create >> FIRST, then identify with Topology
        topo = Topology(tx.tx, self.dst.INP)
        with ReferencePlanes(self.GND):
            self.dp_cst = ConstrainDiffPair(topo).timing_difference(5e-12)
```

### PCIe pattern: lane providers + per-lane constraints

```python
link = self.switch.require(PCIeLink)

dp_cst = DiffPairConstraint(skew=Toleranced(0, 5e-12), loss=3.0)

with ReferencePlanes(self.GND):
    for i in range(num_lanes):
        self += link.lane[i].TX.p >> self.endpoints[i].INP.p
        self += link.lane[i].TX.n >> self.endpoints[i].INP.n
        dp_cst.constrain(link.lane[i].TX, self.endpoints[i].INP)
```

### DDR4 pattern: byte swap + DQ-to-DQS matching

```python
data = self.controller.require(DDR4Data)

with ReferencePlanes(self.GND):
    for bl in range(2):
        offset = bl * bits_per_lane

        # DQS topology (reference signal for matching)
        self += data.byte_lane[bl].DQS.p >> self.mem.DQS_P[bl]
        self += data.byte_lane[bl].DQS.n >> self.mem.DQS_N[bl]
        dqs_topo = Topology(data.byte_lane[bl].DQS.p, self.mem.DQS_P[bl])

        # DQ topologies
        dq_topos = []
        for i in range(bits_per_lane):
            self += data.byte_lane[bl].DQ[i] >> self.mem.DQ[offset + i]
            dq_topos.append(
                Topology(data.byte_lane[bl].DQ[i], self.mem.DQ[offset + i])
            )

        # Match DQ to DQS within 20ps per byte lane
        ConstrainReferenceDifference(
            guide=dqs_topo,
            topologies=dq_topos,
        ).timing_difference(Toleranced(0, 20e-12))
```

## PCIe Lane Flexibility

Model PCIe width variants and lane flexibility using the constructor `Provide` API. The component has individual P/N pins per lane; the Provide mapping connects them to DiffPair bundle sub-ports.

```python
class PCIeSwitchCircuit(Circuit):
    def __init__(self):
        self.sw = PCIeSwitchComponent()

        # x4 link using all 4 lanes
        self.pcie_x4 = Provide(PCIeLink4).one_of(
            lambda b: [self._create_mapping(b, lane_offset=0)]
        )

        # x2 links: lanes 0-1 or lanes 2-3
        self.pcie_x2 = Provide(PCIeLink2).one_of(
            lambda b: [
                self._create_mapping(b, lane_offset=0),
                self._create_mapping(b, lane_offset=2),
            ]
        )

    def _create_mapping(self, b, lane_offset: int) -> dict:
        mapping = {}
        for i in range(len(b.lane)):
            mapping[b.lane[i].TX.p] = self.sw.PTXP[i + lane_offset]
            mapping[b.lane[i].TX.n] = self.sw.PTXN[i + lane_offset]
            mapping[b.lane[i].RX.p] = self.sw.PRXP[i + lane_offset]
            mapping[b.lane[i].RX.n] = self.sw.PRXN[i + lane_offset]
        return mapping
```

## DDR4 Byte/Bit Swapping

DDR4 controllers often support byte lane reordering and bit swapping within a byte lane. Model with `Provide().one_of()`:

```python
class DDR4ControllerCircuit(Circuit):
    def __init__(self):
        self.ctrl = DDR4ControllerComponent()

        self.data_provide = Provide(DDR4Data).one_of(
            lambda b: [
                self._create_mapping(b, byte_swap=False),
                self._create_mapping(b, byte_swap=True),
            ]
        )

    def _create_mapping(self, b: DDR4Data, byte_swap: bool) -> dict:
        mapping = {}
        phys = [1, 0] if byte_swap else [0, 1]
        for logical in range(2):
            p = phys[logical]
            for i in range(8):
                mapping[b.byte_lane[logical].DQ[i]] = self.ctrl.DQ[p * 8 + i]
            mapping[b.byte_lane[logical].DQS.p] = self.ctrl.DQS_P[p]
            mapping[b.byte_lane[logical].DQS.n] = self.ctrl.DQS_N[p]
            mapping[b.byte_lane[logical].DM] = self.ctrl.DM[p]
        return mapping
```

**Important rules:**
- DQS must stay with its byte lane (DQS0 always with byte lane 0's DQ bits)
- Address and command signals are NOT swappable — use fixed wiring
- CK polarity is NOT swappable — use fixed wiring

For protocol-specific swap rules and constraint parameters, see [references/protocol-pin-flexibility.md](references/protocol-pin-flexibility.md).

## Common Mistakes

```python
# WRONG: Returning the port directly instead of a mapping list
@provide(GPIO)
def provide_gpio(self, g: GPIO):
    return self.mcu.GPIO[0]  # WRONG: must return list of dicts

# CORRECT: Return list of {bundle_port: component_port} dicts
@provide(GPIO)
def provide_gpio(self, g: GPIO):
    return [{g.gpio: self.mcu.GPIO[0]}]

# WRONG: Using = to connect required ports (Python assignment)
gpio = self.mcu.require(GPIO)
self.led_pin = gpio.gpio  # WRONG: assignment, not connection

# CORRECT: Use + to create a net
self.led_net = gpio.gpio + self.led.anode

# WRONG: Using @provide.one_of when you want multiple instances
@provide.one_of(GPIO)  # Only gives ONE GPIO total
def provide_gpio(self, g: GPIO):
    return [{g.gpio: p} for p in self.mcu.GPIO]

# CORRECT: Use @provide (all_of) for multiple independent instances
@provide(GPIO)
def provide_gpio(self, g: GPIO):
    return [{g.gpio: p} for p in self.mcu.GPIO]

# WRONG: Topology not stored (silently dropped)
tx = self.src.require(TXLink)
tx.tx.p >> self.dst.INP.p  # BAD: topology lost!

# CORRECT: Store with +=
self += tx.tx.p >> self.dst.INP.p

# WRONG: Creating Topology before creating >> connection
tx = self.src.require(TXLink)
topo = Topology(tx.tx, self.dst.INP)  # Path doesn't exist yet!
self += tx.tx.p >> self.dst.INP.p

# CORRECT: Create >> first, then identify with Topology
self += tx.tx.p >> self.dst.INP.p
self += tx.tx.n >> self.dst.INP.n
topo = Topology(tx.tx, self.dst.INP)

# WRONG: Modeling P/N swap for a protocol that doesn't support it
# DDR4 DQS polarity is FIXED — never swap
@provide.one_of(DDR4DQS)
def provide_dqs(self, dqs: DDR4DQS):
    return [
        {dqs.p: self.ctrl.DQS_P, dqs.n: self.ctrl.DQS_N},
        {dqs.p: self.ctrl.DQS_N, dqs.n: self.ctrl.DQS_P},  # WRONG: not swappable
    ]
```

## Verification

### Step 1: Type Check
```bash
pyright path/to/circuit.py
```

### Step 2: Build Test
```bash
python -m jitx build <module.path.DesignClass>
```

Pin assignment errors appear as "Unsatisfiable pin assignment" in the Issues List. Constraint violations appear under "Unsatisfied Signal Constraints".

## API Reference

For complete class definitions, all parameters, and method signatures:
- [Pin assignment concepts](https://docs.jitx.com/en/latest/essentials/design/pin_assignment.html)
- [jitx.net module (provide, Provide)](https://docs.jitx.com/en/latest/api/jitx.net.html)
- [jitx.si module (constraints)](https://docs.jitx.com/en/latest/api/jitx.si.html)

For protocol-specific pin flexibility rules, see [references/protocol-pin-flexibility.md](references/protocol-pin-flexibility.md).

## Formatting

```bash
ruff format path/to/circuit.py
```
