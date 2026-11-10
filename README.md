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

The diagram shows one possible XLeRobot layout. The SBC receives power and connects to the control network through its own XT30(2+2) connection to the base board, then controls the robot modules. The Feetech boards locally drive their attached servos.

![XLeRobot architecture](renders/xlerobot_architecture.png)

The base board provides six fused outputs, so these four modules use four separate outputs and leave two available. Each branch carries the same CAN bus, so connect only the two bus endpoints with termination enabled.

## Example: AMR with DDSM115 drive and RobStride arm

This AMR layout uses one mod_ddsm for four independently connected DDSM115 wheel motors, with the reBot Arm B601-DM and an IMU on separate base-board branches. The SBC connects through XT30(2+2) for power and control. The arm uses RobStride servos. Check the arm's connection and power requirements against its documentation before wiring.

![AMR architecture with DDSM115 drive and reBot Arm B601-DM](renders/amr_architecture.png)

The base outputs are drawn as separate branches for clarity. They remain electrically connected to the same power source and CAN network; the output fuses isolate branch faults, not CAN traffic. Check motor and arm peak currents against the battery, BMS, wiring, and branch fuse ratings.

## Repository

Each board directory contains KiCad project, schematic, and PCB layout files. Available board renders are shown above and collected in [renders](renders/).

This README describes the current hardware designs. A common CAN protocol is an intended system feature; firmware and a protocol specification are not currently included here.
