# Common Ground

Modular robot electronics built around a shared power and CAN backbone. Combine the boards needed for a robot and chain them through XT30(2+2) connectors. Each board uses a Seeed Studio XIAO RA4M1 to handle its local hardware and is intended to communicate through a common CAN protocol.

## Shared architecture

- **Power and data together:** the XT30(2+2) connections carry supply power, ground, CAN high, and CAN low.
- **Local control:** each module has a XIAO RA4M1, an SN65HVD230 CAN transceiver, and a local 5 V buck regulator.
- **One shared bus:** the connectors extend the same CAN network; they are not independent CAN channels. The boards include jumper-selectable 120 Ω termination, which should be enabled only at the two ends of the bus.

## Boards

| Board | Role and hardware |
| --- | --- |
| [mod_base](mod_base/) | ![mod_base PCB render](renders/mod_base.png)<br>Power entry and distribution hub. Provides an XT60 battery input, a charger/dock screw-terminal input, and **six individually fused XT30(2+2) outputs**. An INA3221 monitors battery, dock, and load currents. Also exposes UART and RS-485 interfaces. |
| [mod_ddsm](mod_ddsm/) | ![mod_ddsm PCB render](renders/mod_hub_motor.png)<br>Hub motor interface (`mod_hub_motor` KiCad project). Provides four motor power outputs, four RS-485 motor connections, and four UART motor connections labelled FL/FR/BL/BR. Includes an SP3485 RS-485 transceiver, a TMUX1308 receive multiplexer, INA226 current monitoring, and a MOSFET power cutoff. Two XT30(2+2) connectors connect it to the backbone. |
| [mod_feetech](mod_feetech/) | ![mod_feetech PCB render](renders/mod_feetech.png)<br>Feetech serial servo interface. Six three-pin connectors share one half-duplex data bus and a regulated servo supply from a TPS56A37 converter. Includes transmit/receive buffers, INA226 current monitoring, a MOSFET power cutoff, and two XT30(2+2) backbone connectors. |
| [mod_sense](mod_sense/) | Motion and heading sensor module. Combines an LSM6DSOX six-axis IMU with an MMC5983MA magnetometer over I²C, with sensor interrupt signals. Has two XT30(2+2) backbone connectors. |
| [mod_kitchensink](mod_kitchensink/mod_kitchensink/) | General I/O and shield-interface board. Provides Arduino-style power, digital/PWM, and analog headers, level translation, an ADS1015 ADC, and a Qwiic-style I²C connector. The current design has **one XT30(2+2) connector**, making it an endpoint rather than a pass-through module. |
| [mod_raw](mod_raw/) | Minimal starting point for new modules. Contains the shared XIAO/CAN/power circuitry, two XT30(2+2) connectors, INA226 current monitoring, and a MOSFET power cutoff, without a dedicated peripheral interface. |

## Repository

Each board directory contains its KiCad project (`.kicad_pro`), schematic (`.kicad_sch`), and PCB layout (`.kicad_pcb`). Board renders are collected in [renders](renders/).

This README describes the current hardware designs. The common CAN protocol is an intended system feature; this repository does not currently contain firmware or a protocol specification.
