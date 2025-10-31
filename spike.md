# Spike Filter Module Expert Guide

## Beginner

### Purpose

The Spike Filter Module suppresses narrow spikes on SDA and SCL inputs to ensure reliable I2C communication. It is instantiated twice: once for SDA and once for SCL.

### Key Points

- Filters spikes based on configurable pulse width.
- Operates only when IP and filter are enabled.

### Extracted Overview

3.6.1	Interface signals	72
3.6.2	Functional Description	72
Interface signals
Table 16: Interface signals – Spike filter Module
Functional Description
The description of different signals used in this design is explained below.
“hs_xfer_in_progress”: This signal is OR of the inputs “hs_xfer_enabled” and “tgt_filter_en_for_hs”.
“pw_reg[3:0]”: This is an internal signal and mux output of logic which selects between “spike_pw_reg” and “hs_spike_pw_reg”. When “hs_xfer_in_progress” is 1’b1 then this signal is “hs_spike_pw_reg” else this signal is “spike_pw_reg”.
“spike_pw_cnt”: This is a 4bit up-down counter with reset value of 0. The configuration register “pw_reg” value should be loaded only on the rising edge of “i2c_en”. This counter should update only when the inputs “i2c_en” and “filter_en” both are 1’b1.
When “filter_in” is 1’b1 then,
the counter increments when the counter is less than or equal to “pw_reg”.
when the counter is “pw_reg + 1” then it retains this value
When “filter_in” is 1’b0 then,
the counter decrements when the counter is greater than 0
when the counter is 0, then it retains the 0 value
“filter_out”: The is sequential internal signal with reset value of 1’b1. This signal will be same as input only when “filter_en” is 1’b0. When “filter_en” is 1’b1 then this signal will transition from:
1’b1 to 1’b0 only when “spike_pw_cnt” is 0,
1’b0 to 1’b1 only when “spike_pw_cnt” is greater than “pw_reg”.
Some important signals are captured in the timing diagram for Spike filter design in Figure 43.
Figure 44: Timing Diagram – Spike Filter for SDA input

## Intermediate

### Relationships with Other Modules

- **Receive Module**: Uses filtered signals (`sda_in`, `scl_in`) for edge detection.

- **Timing Generator**: Coordinates HS mode timing with spike filter thresholds.

- **Transmit Module**: Relies on clean inputs for arbitration and synchronization.

- **CSR Module**: Provides configuration registers for enabling filter and setting thresholds.

### Programming Steps

1. Configure `spike_pw_reg` and `hs_spike_pw_reg`.
2. Enable `i2c_en` and `filter_en`.
3. Ensure HS mode indicators are set correctly.

### Extracted Related Module Notes

3.2.1	Interface signals	27
3.2.2	Functional Description	30
3.3.1	Interface signals	49
3.3.2	Functional Description	50
3.4.1	Interface signals	57
3.4.2	Functional Description	58
3.5.1	Interface signals	68
3.5.2	Functional Description	69
3.6.1	Interface signals	72
3.6.2	Functional Description	72
3.8.1	Interface signals	75
3.9.1	Interface signals	78
3.10.1	Interface signals	81
3.10.2	Functional Description	81
3.11.1	Interface signals	83
3.11.2	Functional Description	84
Table 3: Interface signals of Transmit Controller	17
Table 6: Interface signals of Transmit Module	30
Table 9: Interface signals of Receive Module	50
Table 12: Interface signals of Receiver Controller Module	58
Table 15: Interface signals of Timing Generator Module	69
Table 18: Interface signals - Register Module	78
Table 19: Interface signals – FIFO Module	80
Table 20: Interface signals – DMA Interface Module	81
Table 21: Interface signals – APB Slave Module	83
The Top IP consists of standalone I2C IP integrated with APB Slave Interface Module. Table 1 lists the Interface signals of the I2C IP with APB Interface Wrapper.
The standalone I2C IP will have a generic Register Interface which can be used to access IP’s Control and Status Registers as well as the Transmit and Register FIFO. Table 2 lists the Interface signals of the standalone I2C IP. The APB Slave Interface module will be integrated with standalone I2C IP. Address Decoder logic will be part of the APB Slave Interface which will generate the write and read enables. The CSR Interface listed in Table 2 is described in detail in Section 3.11.
Interface signals
Table 3: Interface signals of Transmit Controller
Functional Description
Interface signals
: Interface signals of Transmit Module
Functional Description
Interface signals
Table 9: Interface signals of Receive Module
Functional Description
Interface signals
Table 12: Interface signals of Receiver Controller Module
Functional Description
Interface signals
Table 15: Interface signals of Timing Generator Module
Functional Description
Interface signals
Functional Description
Interface signals
Table 18: Interface signals - Register Module
Interface signals
Table 19: Interface signals – FIFO Module
Interface signals
Table 20: Interface signals – DMA Interface Module
Functional Description
Interface signals
Table 21: Interface signals – APB Slave Module
Functional Description

## Advanced

### Internal Logic Details

- HS mode detection via `hs_xfer_in_progress`.
- Threshold selection using `pw_reg` mux.
- Up/down counter (`spike_pw_cnt`) increments/decrements based on input level.
- Output transition rules:
  - 1→0 only when counter == 0.
  - 0→1 only when counter > pw_reg.

### Edge Cases & Expert Tips

- Reset behavior: `filter_out` defaults to 1.
- Bypass mode when `filter_en` = 0.
- Configure thresholds before enabling IP.

### Detailed Signal Description Table

| Signal | Direction | Width | Domain | Reset | Valid Conditions | Description |

|--------|-----------|-------|--------|-------|------------------|-------------|

| Table 17: Interrupt mapping	75 |

| Table 18: Interface signals - Register Module	78 |

| Table 19: Interface signals – FIFO Module	80 |

| Table 20: Interface signals – DMA Interface Module	81 |

| Table 21: Interface signals – APB Slave Module	83 |

| Table 22: Configuration and Status Register List	86 |

| Table 23: Header Fields present in Packet processed by Controller	100 |

| Table 24: Packet Formal for General Call Address – Case1	101 |

| Table 25: Packet Formal for General Call Address – Case2	102 |

| Table 26: Compile time Defines	103 |

| Table 27: Design Parameters	104 |

| Table 28: Ram Details	106 |

| Table 29: DFD Signal Description	108 |

| Table 30 Latency Calculation	111 |

| Introduction |

| Identification |

| Requirement |

| I2C IP with APB interface to support the standard I2C Bus Protocol and APB5 Interface on the Application side. Different features supported by the IP are listed in Section 1.5. |

| References |

| I2C Specification, Rev7.0, 01/Oct/2021 (https://www.nxp.com/docs/en/user-guide/UM10204.pdf) |

| AMBA APB Protocol Specification, Version D, 09/Apr/2021 |

| IP Licensing |

| TBD |

| Features |

| Following features are supported by the IP: |

| Configuration: Controller and Target |

| Programmable via Host interface |

| Only one configuration will be active at a given time |

| Different bus modes: |

| Standard Mode (0 to 100 kbit/s) |

| Fast Mode (≤ 400 kbit/s) |

| Fast Mode Plus (≤ 1000 kbit/s) |

| High Speed Mode (≤ 3.4 Mbit/s) |

| Target Addressing: 7bit or 10bit |

| Device ID support by Controller and Target (request and response generated in Fast or Standard mode). |

| Start Byte, Software Reset and General Call Address support by Controller and Target |

| Clock Synchronization and Arbitration by Controller |

| Clock stretching support by Target |

| Level or Edge Interrupt defined via parameter |

| Standard APB5 Interface used for Host interface |

| Separate clock domain for IP Core functionality and Host Interface. |

| Parameter used to select same clock or different clocks for IP Core functionality and Host Interface. |

| Buffers on both Transmit and Receive path with parameterizable depths. |

| DMA interface for Host to DMA data into/from the IP Buffers. |

| Spike suppression on SDA and SCL inputs. |

| Assumptions |

| None |

| Limitations |

| This IP does not support following features: |

| Ultra Fast Mode feature |

| Bus Clear command |

| Opens |

| None |

| Design Overview |

| Top Level block diagram is shown in Figure 1 gives an overview on the various interfaces and blocks of the IP. The host interface is kept external to the I2C IP. This way I2C IP can be quickly integrated with required host interface. In the current release, APB Slave interface integrates with the I2C IP. |

| Figure 1: Architecture Block Diagram |

| Interface Signals |

| The Top IP consists of standalone I2C IP integrated with APB Slave Interface Module. Table 1 lists the Interface signals of the I2C IP with APB Interface Wrapper. |

| Table 1: Interface Signals of I2C IP with APB Interface Wrapper |

| The standalone I2C IP will have a generic Register Interface which can be used to access IP’s Control and Status Registers as well as the Transmit and Register FIFO. Table 2 lists the Interface signals of the standalone I2C IP. The APB Slave Interface module will be integrated with standalone I2C IP. Address Decoder logic will be part of the APB Slave Interface which will generate the write and read enables. The CSR Interface listed in Table 2 is described in detail in Section 3.11. |

| Table 2: Interface Signals of standalone I2C IP |

| Interface Timing Diagram |

| APB Write Transaction |

| Refer Figure 44 for APB Write Transaction seen on APB Interface. |

| APB Read Transaction |

| Refer Figure 45 for APB Read Transaction seen on APB Interface |

| Detailed Description |

| The I2C module consists of following sub-blocks. |

| Transmit Controller Module |

| Transmit Module |

| Receive Module |

| Receive Controller Module |

| Timing Generator Module |

| Interrupt Module |

| Control and Status Register Module |

| Asynchronous FIFO Module |

| DMA Interface Module |

| APB Slave Module |

| Reset Generation Module. This is covered in Section 6.1. |

| Transmit Controller Module |

| Transmit Controller sits between the Transmit FIFO and Transmit Module. It reads from the Transmit FIFO whenever it is not empty and provides the 32bit data to the Transmit Module. It maintains Finite State Machine (FSM) which updates based on the transaction being processed. The Transmit controller has two operating modes: Controller and Target mode. |

| Interface signals |

| Table 3: Interface signals of Transmit Controller |
