# Common Ground

Modular robot electronics built around a shared power and CAN backbone. Modules chain through XT30(2+2) connectors. Each module has a Seeed Studio XIAO RA4M1 for local control and is intended to communicate over a common CAN protocol.

## Shared architecture

- XT30(2+2) carries supply power, ground, CAN high, and CAN low.
- The XIAO RA4M1 and SN65HVD230 CAN transceiver provide local control and bus access.
- Boards include jumper-selectable 120 Ω CAN termination. Enable termination only at the two physical ends of a bus.

## Boards

| Board | Render | Role and hardware |
| --- | --- | --- |
| [mod_base](mod_base/) | <img src="renders/mod_base.png" alt="mod_base PCB" width="210"> | - Battery input via XT60; charger/dock input via screw terminal<br>- Individually fused XT30(2+2) power/CAN outputs<br>- INA3221 monitors battery, dock, and load current<br>- UART and RS-485 interfaces |
| [mod_ddsm](mod_ddsm/) | <img src="renders/mod_hub_motor.png" alt="mod_ddsm PCB" width="210"> | - Four hub motor power outputs<br>- Four RS-485 and four UART connections, labelled FL/FR/BL/BR<br>- SP3485, TMUX1308 receive multiplexer, INA226 current monitor, and MOSFET power cutoff<br>- Two XT30(2+2) backbone connectors |
| [mod_feetech](mod_feetech/) | <img src="renders/mod_feetech.png" alt="mod_feetech PCB" width="210"> | - Six three-pin Feetech servo connectors on a shared half-duplex data bus<br>- 8 A servo supply regulator (TPS56A37)<br>- Transmit/receive buffers, INA226 current monitor, and MOSFET power cutoff<br>- Two XT30(2+2) backbone connectors |
| [mod_sense](mod_sense/) | <img src="renders/mod_sense.png" alt="mod_sense PCB" width="210"> | - LSM6DSOX six-axis IMU and MMC5983MA magnetometer over I²C<br>- Sensor interrupt signals<br>- Two XT30(2+2) backbone connectors |
| [mod_kitchensink](mod_kitchensink/mod_kitchensink/) | <img src="renders/mod_kitchensink.png" alt="mod_kitchensink PCB" width="210"> | - Arduino-style power, digital/PWM, and analog headers<br>- Level shifters for compatibility with virtually every Arduino shield |
| [mod_raw](mod_raw/) | — | - Minimal starting point for custom modules<br>- Shared XIAO/CAN/power circuitry, INA226 current monitor, and MOSFET power cutoff |

## Example: XLeRobot module chain

The diagram illustrates one possible physical daisy chain from separate fused outputs on the base board. The SBC is the robot's high-level computer; it would need a CAN interface to join the bus. All modules share the same CAN bus; each Feetech board locally controls its attached servos.

```mermaid
flowchart LR
    BAT[Battery] --> BMS[BMS] --> BASE[mod_base]
    BASE -->|Output 1 · XT30 power + CAN| FEET_BASE[Feetech<br/>Base motion<br/>3 servos]
    BASE -->|Output 2 · XT30 power + CAN| SENSE[mod_sense<br/>IMU]
    BASE -->|Output 3 · XT30 power + CAN| FEET_LEFT[Feetech<br/>Left arm<br/>6 servos]
    BASE -->|Output 4 · XT30 power + CAN| FEET_RIGHT[Feetech<br/>Right arm<br/>6 servos]
    SBC[SBC<br/>High-level control] <-->|CAN interface| FEET_BASE

    classDef base fill:#f6bd60,stroke:#9a5b00,color:#202020,stroke-width:2px;
    classDef feetech fill:#90caf9,stroke:#1565c0,color:#102a43,stroke-width:2px;
    classDef sense fill:#a5d6a7,stroke:#2e7d32,color:#17351a,stroke-width:2px;
    class BAT,BMS base;
    class BASE base;
    class FEET_BASE,FEET_LEFT,FEET_RIGHT feetech;
    class SENSE sense;
```

The base board provides six fused outputs, so these four modules use four separate outputs and leave two available. Each branch carries the same CAN bus, so connect only the two bus endpoints with termination enabled.

## Example: AMR with DDSM115 drive and RobStride arm

This layout uses one mod_ddsm for four independently connected DDSM115 wheel motors. The arm's RobStride servos connect directly to a separate XT30 branch; they do not pass through a Feetech board. Since the board's XT30(2+2) pinout assigns the other two contacts to CANH and CANL, confirm that the RobStride model uses a compatible connector pinout and bus signaling before connecting it. If it does, the SBC can communicate on the shared CAN bus through its CAN interface. The SBC also handles AMR navigation and higher-level control.

```mermaid
flowchart LR
    BAT[Battery] --> BMS[BMS] --> BASE[mod_base]
    BASE -->|Output 1 · XT30 power + CAN| DDSM[mod_ddsm<br/>Motor controller]
    BASE -->|Output 2 · XT30 power + CAN| ARM[RobStride arm<br/>Servos connect directly]
    BASE -->|Output 3 · XT30 power + CAN| SENSE[mod_sense<br/>IMU]
    SBC[SBC<br/>Navigation + high-level control] <-->|CAN interface| DDSM

    DDSM -->|Motor power + RS-485| M1[DDSM115<br/>Front left]
    DDSM -->|Motor power + RS-485| M2[DDSM115<br/>Front right]
    DDSM -->|Motor power + RS-485| M3[DDSM115<br/>Rear left]
    DDSM -->|Motor power + RS-485| M4[DDSM115<br/>Rear right]

    classDef base fill:#f6bd60,stroke:#9a5b00,color:#202020,stroke-width:2px;
    classDef ddsm fill:#ce93d8,stroke:#6a1b9a,color:#26102d,stroke-width:2px;
    classDef arm fill:#ffab91,stroke:#bf360c,color:#3e160b,stroke-width:2px;
    classDef sense fill:#a5d6a7,stroke:#2e7d32,color:#17351a,stroke-width:2px;
    classDef motor fill:#b0bec5,stroke:#455a64,color:#182329,stroke-width:2px;
    class BAT,BMS,BASE base;
    class DDSM ddsm;
    class ARM arm;
    class SENSE sense;
    class M1,M2,M3,M4 motor;
```

The base outputs are drawn as separate branches for clarity. They remain electrically connected to the same power source and CAN network; the output fuses isolate branch faults, not CAN traffic. Check motor and arm peak currents against the battery, BMS, wiring, and branch fuse ratings.

## Repository

Each board directory contains KiCad project, schematic, and PCB layout files. Available board renders are shown above and collected in [renders](renders/).

This README describes the current hardware designs. A common CAN protocol is an intended system feature; firmware and a protocol specification are not currently included here.
