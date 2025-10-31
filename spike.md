# Spike Filter Module — Expert Guide
## Beginner
### Purpose
- Suppresses narrow spikes on SDA and SCL inputs before they reach the Receive Module.
- Ensures reliable edge detection for START/STOP conditions.

### Key Signals Overview
- `filter_in`: Raw asynchronous input from pad.
- `filter_out`: Filtered and synchronized output to Receive Module.
- `i2c_en`: IP enable; filter works only when asserted.
- `filter_en`: Enables spike suppression logic.

## Intermediate
### Relationships with Other Modules
- **Receive Module**: Consumes `filter_out` for SDA/SCL edge detection.
- **Timing Generator**: Coordinates HS vs Standard timing; HS mode affects spike width threshold.
- **Transmit Module**: Relies on clean inputs for arbitration and clock sync.
- **CSR Module**: Provides configuration registers (`spike_pw_reg`, `hs_spike_pw_reg`).

### Programming Steps
1. Configure `spike_pw_reg` and `hs_spike_pw_reg`.
2. Enable `i2c_en`.
3. Enable `filter_en`.

## Advanced
### Internal Logic
- `hs_xfer_in_progress` = OR of `hs_xfer_enabled` and `tgt_filter_en_for_hs`.
- `pw_reg` selects between `spike_pw_reg` and `hs_spike_pw_reg`.
- `spike_pw_cnt` is a 4-bit up/down counter:
  - Increments when `filter_in=1` until `pw_reg+1`.
  - Decrements when `filter_in=0` until 0.
- `filter_out` transitions:
  - 1→0 only when `spike_pw_cnt==0`.
  - 0→1 only when `spike_pw_cnt > pw_reg`.

### Detailed Signal Description Table
| Signal Name | Direction | Width | Domain | Reset | Valid Conditions | Description | Source/Destination |
|-------------|-----------|-------|--------|-------|-------------------|-------------|---------------------|
| clk | In | 1 | “core_clk” connected if parameter SINGLE_CLK_DOMAIN = 0
“pclk” connected if parameter SINGLE_CLK_DOMAIN = 1 |  |  |  |  |
| rst_n | In | 1 | Asynchronous reset with rising edge synchronized to appropriate  clock input to the logic. Use “i2c_ctrl_rst_n” as described in Section 6.1 |  |  |  |  |
| i2c_en | In | 1 | As described in Table 18. |  |  |  |  |
| ip_mode | In | 1 | As described in Table 18. |  |  |  |  |
| ip_addr_mode | In | 1 | As described in Table 18. |  |  |  |  |
| hs_mode_en | In | 1 | As described in Table 18. |  |  |  |  |
| request_device_id_reg | In | 1 | As described in Table 18. |  |  |  |  |
| tgt_addr_is_10bit | In | 1 | As described in Table 18. |  |  |  |  |
| tgt_addr_reg[9:0] | In | 10 | As described in Table 18. |  |  |  |  |
|  |  |  | Table 18 |  |  |  |  |
| ip_in_controller_mode | Out | 1 | Static signal indicating if IP is operating in Controller mode |  |  |  |  |
| ip_in_target_mode | Out | 1 | Stati signal if IP is operating in Target Mode |  |  |  |  |
| ff_nempty | In | 1 | FIFO Not Empty status from the Transmit FIFO |  |  |  |  |
| ff_rvld | In | 1 | Read Valid signal accompany the Read data from Transmit FIFO |  |  |  |  |
| ff_rdata | In | DATAW | Read data from Transmit FIFO. The Parameter DATAW will be 32 for Controller Mode and 8 for Transmit Mode. |  |  |  |  |
| ff_ren | Out | 1 | Read Enable output to Transmit FIFO |  |  |  |  |
| xmt_rdy | In | 1 | As described in Table 6. |  |  |  |  |
| ctl_in_rcv_mode | In | 1 | As described in Table 6. |  |  |  |  |
| hs_cmd_being_sent | In | 1 | As described in Table 6. |  |  |  |  |
| start_byte_being_sent | In | 1 | As described in Table 6. |  |  |  |  |
| xmt_req | Out | 1 | Request signal to Transmit Module validating the “cmd_data” |  |  |  |  |
| cmd_data | Out | DATAW | Command or Data sent to Transmit Module. The Parameter DATAW will be 32 for Controller Mode and 8 for Transmit Mode. |  |  |  |  |
| last_byte | Out | 1 | Signal indicating last byte transaction (write or read) is pending. |  |  |  |  |
| send_restart | Out | 1 | Used in Controller mode. Sent to Transmit module for it to transmit Restart condition. |  |  |  |  |
| send_ack | Out | 1 | Used in Controller mode. Sent to Transmit module for it to transmit ACK condition. |  |  |  |  |
| scL_fe | In | 1 | As described in Table 9 |  |  |  |  |
| rcv_ack | In | 1 | As described in Table 9 |  |  |  |  |
| rcv_nack | In | 1 | As described in Table 9 |  |  |  |  |
| arb_lost | In | 1 | s described in Table 6. |  |  |  |  |
| rcv_vld | In | 1 | As described in Table 9 |  |  |  |  |
| sending_dev_id_req | Out | 1 | Used in Controller mode. Sent to Receive Controller module to receive the Target Device ID for Device ID request being sent. |  |  |  |  |
| bytes_not_sent | Out | 8 | Status information indicating number of write data yet to be transmitted before receiving NACK from Target for Write command. |  |  |  |  |
| target_addr_sts | Out | 10 | Target Address from which NACK was received. |  |  |  |  |
| xmt_ctrl_state[3:0] | Out | 4 | This signal is the “pstate” of this module. This is used as Status information |  |  |  |  |
| nack_rcvd_intr | Out | 1 | Interrupt indicating NACK received from Target |  |  |  |  |
| dev_id_req_cmpl | Out | 1 | This signal indicates successful completion of Device ID request. |  |  |  |  |
| dev_id_req_nack | Out | 1 | This signal indicates the Device ID request has been NACK’d by the Target Devices on the bus. |  |  |  |  |
| Signal Name | In/
Out | Width (bits) | Description |  |  |  |  |
| clk | In | 1 | “core_clk” connected if parameter SINGLE_CLK_DOMAIN = 0
“pclk” connected if parameter SINGLE_CLK_DOMAIN = 1 |  |  |  |  |
| rst_n | In | 1 | Asynchronous reset with rising edge synchronized to appropriate  clock input to the logic. Use “i2c_ctrl_rst_n” as described in Section 6.1 |  |  |  |  |
| i2c_en | In | 1 | As described in Table 18. |  |  |  |  |
| ip_mode | In | 1 | As described in Table 18. |  |  |  |  |
| ip_addr_mode | In | 1 | As described in Table 18. |  |  |  |  |
| en_clk_stretch | In | 1 | As described in Table 18. |  |  |  |  |
| send_start_byte | In | 1 | As described in Table 18. |  |  |  |  |
| multi_ctrl_en | In | 1 | As described in Table 18. |  |  |  |  |
| tgt_revision[2:0] | In | 3 | As described in Table 18. |  |  |  |  |
| tgt_part_id[8:0] | In | 9 | As described in Table 18. |  |  |  |  |
| tgt_manufacturer[11:0] | In | 12 | As described in Table 18. |  |  |  |  |
| tgt_dev_id_valid | In | 1 | As described in Table 18. |  |  |  |  |
| hs_mode_ctrl_addr_reg | In | 3 | As described in Table 18. |  |  |  |  |
| hs_mode_en | In | 1 | As described in Table 18. |  |  |  |  |
| ip_in_controller_mode | In | 1 | As described in Table 3. |  |  |  |  |
| ip_in_target_mode | In | 1 | As described in Table 3. |  |  |  |  |
| xmt_req | In | 1 | As described in Table 3. |  |  |  |  |
| cmd_data | In | DATAW | As described in Table 3. |  |  |  |  |
| last_byte | In | 1 | As described in Table 3. |  |  |  |  |
| send_restart | In | 1 | As described in Table 3. |  |  |  |  |
| send_ack | In | 1 | As described in Table 3. |  |  |  |  |
| rcv_ack | In | 1 | As described in Table 9 |  |  |  |  |
| rcv_nack | In | 1 | As described in Table 9 |  |  |  |  |
| done_sta_hd | In | 1 | As described in Table 15 |  |  |  |  |
| done_sta_su | In | 1 | As described in Table 15 |  |  |  |  |
| done_sda_hd | In | 1 | As described in Table 15 |  |  |  |  |
| done_scl_low | In | 1 | As described in Table 15 |  |  |  |  |
| done_scl_hi | In | 1 | As described in Table 15 |  |  |  |  |
| done_sto_su | In | 1 | As described in Table 15 |  |  |  |  |
| done_tbuf | In | 1 | As described in Table 15 |  |  |  |  |
| done_sda_su | In | 1 | As described in Table 15 |  |  |  |  |
| tgt_rcv_data_st | In | 1 | As described in Table 12 |  |  |  |  |
| tgt_send_data_st | In | 1 | As described in Table 12 |  |  |  |  |
| tgt_send_ack_st | In | 1 | As described in Table 12 |  |  |  |  |
| tgt_ack_st | In | 1 | As described in Table 9 |  |  |  |  |
| scl_re | In | 1 | Signal indicating rising edge of SCL detected by Receiver. Used in Target mode. |  |  |  |  |
| scl_fe | In | 1 | Signal indicating falling edge of SCL detected by Receiver. Used in Target mode. |  |  |  |  |
| sda_in | In | 1 | Synchronized SDA input line (from Filter logic). This is used by Controller during Arbitration phase to check whatever was transmitted has been received. |  |  |  |  |
| scl_in | In | 1 | Synchronized SCL input line (from Filter logic). This is used for Clock Synchronization |  |  |  |  |
| bus_not_idle | In | 1 | As described in Table 9. |  |  |  |  |
| bit_cnt[3:0] | In | 1 | As described in Table 9 |  |  |  |  |
| rx_ff_full | In | 1 | Receive FIFO Full condition. Used in Target mode. |  |  |  |  |
| dev_id_rreq_rcvd | In | 1 | As described in Table 12. |  |  |  |  |
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |
| hs_xfer_enabled | Out | 1 | Signal indicating High Speed transfer is enabled and hence the timing generator should switch to High Speed Timing configuration values. |  |  |  |  |
| xmt_rdy | Out | 1 | Signal indicating Transmit module is Ready or in Idle state |  |  |  |  |
| rd_cmd | Out | 1 | Signal indicating Read Command has been transmitted and expecting response from Target. |  |  |  |  |
| ld_sta_hd | Out | 1 | Pulse sent to Timing generator to load I2C START Hold value in the counter. |  |  |  |  |
| ld_sta_su | Out | 1 | Pulse sent to Timing generator to load I2C START Setup value in the counter. |  |  |  |  |
| ld_scl_low | Out | 1 | Pulse sent to Timing generator to load I2C SCL Low value in the counter. |  |  |  |  |
| ld_scl_hi | Out | 1 | Pulse sent to Timing generator to load I2C SCL High value in the counter. |  |  |  |  |
| ld_sto_su | Out | 1 | Pulse sent to Timing generator to load I2C STOP Setup value in the counter. |  |  |  |  |
| ld_tbuf | Out | 1 | Pulse sent to Timing generator to load I2C TBUF value in the counter. |  |  |  |  |
| ld_sda_su | Out | 1 | Pulse sent to Timing generator to load I2C Data Setup value in the counter. |  |  |  |  |
| ld_sda_hd | Out | 1 | Pulse sent to Timing generator to load I2C Data Hold value in the counter. Used in Target mode. |  |  |  |  |
| ctl_send_stop | Out | 1 | This signal will be 1’b1 when FSM is in STOP state |  |  |  |  |
| ctl_send_rstart | Out | 1 | This signal will be 1’b1 when FSM is in Repeated START state |  |  |  |  |
| ctrl_start_byte_ack_err | Out | 1 | This is an error indicating ACK was received when Controller transmitted Start Byte. Controller will continue with the transmission of current command. This error will be used as an interrupt indication. |  |  |  |  |
| ctrl_hs_cmd_ack_err | Out | 1 | This is an error indicating ACK was received when Controller transmitted High Speed Command. Controller will continue with the transmission of current command. This error will be used as an interrupt indication. |  |  |  |  |
| ctrl_arb_lost | Out | 1 | This is an interrupt signal indicating controller has lost arbitration. This signal drives the “arb_lost_intr” interrupt. |  |  |  |  |
| scl_not_sync | Out | 1 | This signal indicates the SCL line is not synchronized. This signal will be used to halt counting in Timing generator. |  |  |  |  |
| xmt_state[3:0] | Out | 4 | This signal is the “pstate” of this module. This is used as Status information |  |  |  |  |
| sda_oe_n | Out | 1 | Active Low output enable to I2C SDA Pad. When value is
0 := I2C SDA Pad should drive Logic 0
1 := I2C SDA Pad should be tri-stated |  |  |  |  |
| scl_oe_n | Out | 1 | Active Low output enable to I2C SCL Pad. When value is
0 := I2C SCL Pad should drive Logic 0
1 := I2C SCL Pad should be tri-stated |  |  |  |  |
| Signal Name | In/
Out | Width (bits) | Description |  |  |  |  |
| clk | In | 1 | “core_clk” connected if parameter SINGLE_CLK_DOMAIN = 0
“pclk” connected if parameter SINGLE_CLK_DOMAIN = 1 |  |  |  |  |
| rst_n | In | 1 | Asynchronous reset with rising edge synchronized to appropriate  clock input to the logic. Use “i2c_ctrl_rst_n” as described in Section 6.1 |  |  |  |  |
| i2c_en | In | 1 | As described in Table 18. |  |  |  |  |
| ip_mode | In | 1 | As described in Table 18. |  |  |  |  |
| sda_in | In | 1 | I2C SDA Input from Spike Filter instance for SDA |  |  |  |  |
| scl_in | In | 1 | I2C SCL Input from Spike Filter instance for SCL |  |  |  |  |
| ctl_xmt_busy | In | 1 | As described in Table 6 |  |  |  |  |
| ctl_in_rcv_mode | In | 1 | As described in Table 6 |  |  |  |  |
| tgt_hs_cmd_rcvd | In | 1 | As described in Table 12 |  |  |  |  |
| tgt_hs_xfer_rcvd | In | 1 | As described in Table 12 |  |  |  |  |
| tgt_start_byte_rcvd | In | 1 | As described in Table 12 |  |  |  |  |
|  |  |  | Table 6 |  |  |  |  |
| rcv_data | Out | 8 | De-serialized data |  |  |  |  |
| rcv_vld | Out | 1 | Pulse validating “rcv_data” |  |  |  |  |
| rcv_ack | Out | 1 | Pulse indicating I2C ACK |  |  |  |  |
| rcv_nack | Out | 1 | Pulse indicating I2C NACK |  |  |  |  |
| eot_detected | Out | 1 | Signal indicating end of transaction is detected. This signal is used in Target mode. |  |  |  |  |
| tgt_ack_st | Out | 1 | Signal indicating Receiver is in ACK/NACK receive state. |  |  |  |  |
| scl_fe | Out | 1 | Pulse indicating falling edge of SCL line |  |  |  |  |
| scl_re | Out | 1 | Pulse indicating rising edge of SCL line |  |  |  |  |
| bus_not_idle | Out | 1 | Signal indicating that the I2C Bus is not IDLE and transactions are seen on receive line. |  |  |  |  |
| bit_cnt[3:0] | Out | 1 | Indicates the number of bits sampled |  |  |  |  |
| restart_rcvd | Out | 1 | Pulse indicating Repeated START received |  |  |  |  |
| bit_cnt_eq8 | Out | 1 | Signal indicating 8bits of data has been received. |  |  |  |  |
| tgt_filter_en_for_hs | Out | 1 | Signal to enable timing generator and spike filter for High Speed mode |  |  |  |  |
| tgt_hs_cmd_ack_err | Out | 1 | Pulse indicating ACK received after the High Speed Command. This is generated by Target. |  |  |  |  |
| tgt_start_byte_ack_err | Out | 1 | Pulse indicating ACK received after Start Byte. This is generated by Target. |  |  |  |  |
| rcv_state[2:0] | Out | 3 | This signal is the “pstate” of this module. This is used as Status information |  |  |  |  |
| Signal Name | In/
Out | Width (bits) | Description |  |  |  |  |
| clk | In | 1 | “core_clk” connected if parameter SINGLE_CLK_DOMAIN = 0
“pclk” connected if parameter SINGLE_CLK_DOMAIN = 1 |  |  |  |  |
| rst_n | In | 1 | Asynchronous reset with rising edge synchronized to appropriate  clock input to the logic. Use “i2c_ctrl_rst_n” as described in Section 6.1 |  |  |  |  |
| i2c_en | In | 1 | As described in Table 18. |  |  |  |  |
| ip_mode | In | 1 | As described in Table 18. |  |  |  |  |
| hs_mode_en | In | 1 | As described in Table 18. |  |  |  |  |
| ip_addr_mode | In | 1 | As described in Table 18. |  |  |  |  |
| en_clk_stretch | In | 1 | As described in Table 18. |  |  |  |  |
| gen_call_addr_en | In | 1 | As described in Table 18. |  |  |  |  |
| target_addr | In | 10 | As described in Table 18. |  |  |  |  |
| tgt_dev_id_valid | In | 1 | As described in Table 18. |  |  |  |  |
| rcv_data | In | 8 | As described in Table 9. |  |  |  |  |
| rcv_vld | In | 1 | As described in Table 9. |  |  |  |  |
| rd_cmd | In | 1 | As described in Table 6. |  |  |  |  |
| rcv_ack | In | 1 | As described in Table 9. |  |  |  |  |
| rcv_nack | In | 1 | As described in Table 9. |  |  |  |  |
| eot_detected | In | 1 | As described in Table 9. Used in Target Mode. |  |  |  |  |
| ff_full | In | 1 | FIFO full indication from Receive FIFO. Used in Target Mode. |  |  |  |  |
| restart_rcvd | In | 1 | As described in Table 9. |  |  |  |  |
| bit_cnt_eq8 | In | 1 | As described in Table 9. |  |  |  |  |
| tgt_dev_id_valid | In | 1 | Signal validating the Target Device ID register. Used in Target mode. |  |  |  |  |
| sending_dev_id_req | In | 1 | As described in Table 3. |  |  |  |  |
| ff_wdata | Out | 8 | Write data to Receive FIFO |  |  |  |  |
| ff_wen | Out | 1 | Write enable to Receive FIFO |  |  |  |  |
| tgt_send_data_st | Out | 1 | State indicating Target is sending read data |  |  |  |  |
| tgt_rcv_data_st | Out | 1 | State indicating Target is receiving write data |  |  |  |  |
| tgt_send_ack_st | Out | 1 | State indicating Target to send ACK on SDA line. |  |  |  |  |
| dev_id_rreq_rcvd | Out | 1 | Signal indicating Device ID request has been received by Target Receiver. This signal will be used by Target Tranmsitter to send Device ID fields. |  |  |  |  |
| tgt_hs_xfer_rcvd | Out | 1 | Signal indicating High Speed transfer in progress. |  |  |  |  |
| tgt_hs_cmd_rcvd | Out | 1 | Signal indicating High Speed command is received. |  |  |  |  |
| tgt_start_byte_rcvd | Out | 1 | Signal indicating Start Byte received by Target. This signal is used by Receive module and is a level signal. |  |  |  |  |
| hs_cmd_ctrl_id_rcvd | Out | 3 | Generated by Target indicating the Controller ID received with High Speed Command. |  |  |  |  |
| hs_cmd_vld | Out | 1 | Pulse validating “hs_cmd_ctrl_id_rcvd”. This is used as interrupt as well as sampling signal to store the “hs_cmd_ctrl_id_rcvd” status. |  |  |  |  |
| manufacturer_id_resp[11:0] | Out | 12 | Manufacturer ID received by Controller for Device ID request |  |  |  |  |
| part_id_resp[8:0] | Out | 9 | Part ID received by Controller for Device ID request |  |  |  |  |
| revision_resp[2:0] | Out | 3 | Revision received by Controller for Device ID request |  |  |  |  |
| rcv_ctrl_state[3:0] | Out | 4 | This signal is the “pstate” of this module. This is used as Status information. |  |  |  |  |
| rd_ack_intr | Out | 1 | Interrupt signal indicating ACK received by Target for data transmitted. This signal is a pulse for 1 clock. |  |  |  |  |
| tgt_start_byte_intr | Out | 1 | Interrupt signal indicating Start Byte received by Target. This signal is a pulse for 1 clock. |  |  |  |  |
| tgt_gen_call_addr_intr | Out | 1 | Interrupt signal indicating General Call Address received by Target. This signal is a pulse for 1 clock. |  |  |  |  |
| Signal Name | In/
Out | Width (bits) | Description |  |  |  |  |
| Clk | In | 1 | “core_clk” connected if parameter SINGLE_CLK_DOMAIN = 0
“pclk” connected if parameter SINGLE_CLK_DOMAIN = 1 |  |  |  |  |
| rst_n | In | 1 | Asynchronous reset with rising edge synchronized to appropriate  clock input to the logic. Use “i2c_ctrl_rst_n” as described in Section 6.1 |  |  |  |  |
| i2c_en | In | 1 | As described in Table 18. |  |  |  |  |
| ip_mode | In | 1 | As described in Table 18. |  |  |  |  |
| ld_sta_hd | In | 1 | As described in Table 6. |  |  |  |  |
| ld_sta_su | In | 1 | As described in Table 6. |  |  |  |  |
| ld_scl_low | In | 1 | As described in Table 6. |  |  |  |  |
| ld_scl_hi | In | 1 | As described in Table 6. |  |  |  |  |
| ld_sto_su | In | 1 | As described in Table 6. |  |  |  |  |
| ld_tbuf | In | 1 | As described in Table 6. |  |  |  |  |
| ld_sda_su | In | 1 | As described in Table 6. |  |  |  |  |
| ld_sda_hd | In | 1 | As described in Table 6. |  |  |  |  |
| sta_hd_reg | In | 16 | As described in Table 18. |  |  |  |  |
| sta_su_reg | In | 16 | As described in Table 18. |  |  |  |  |
| sda_hd_reg | In | 16 | As described in Table 18. |  |  |  |  |
| sda_su_reg | In | 16 | As described in Table 18. |  |  |  |  |
| scl_low_reg | In | 16 | As described in Table 18. |  |  |  |  |
| scl_hi_reg | In | 16 | As described in Table 18. |  |  |  |  |
| sto_su_reg | In | 16 | As described in Table 18. |  |  |  |  |
| tbuf_reg | In | 16 | As described in Table 18. |  |  |  |  |
| hs_sta_hd_reg | In | 16 | As described in Table 18. |  |  |  |  |
| hs_sta_su_reg | In | 16 | As described in Table 18. |  |  |  |  |
| hs_sda_hd_reg | In | 16 | As described in Table 18. |  |  |  |  |
| hs_sda_su_reg | In | 16 | As described in Table 18. |  |  |  |  |
| hs_scl_low_reg | In | 16 | As described in Table 18. |  |  |  |  |
| hs_scl_hi_reg | In | 16 | As described in Table 18. |  |  |  |  |
| hs_sto_su_reg | In | 16 | As described in Table 18. |  |  |  |  |
| hs_xfer_enabled | In | 1 | As described in Table 6. |  |  |  |  |
| scl_not_sync | In | 1 | As described in Table 6. |  |  |  |  |
| tgt_filter_en_for_hs | In | 1 | As described in Table 9. |  |  |  |  |
| done_sta_hd | Out | 1 | Sinal indicating I2C START hold time elapsed. |  |  |  |  |
| done_sta_su | Out | 1 | Sinal indicating I2C START setup time elapsed. |  |  |  |  |
| done_sda_hd | Out | 1 | Sinal indicating SDA hold time elapsed. |  |  |  |  |
| done_scl_low | Out | 1 | Sinal indicating I2C SCL Low time elapsed. |  |  |  |  |
| done_scl_hi | Out | 1 | Sinal indicating I2C SCL High time elapsed. |  |  |  |  |
| done_sto_su | Out | 1 | Sinal indicating I2C Stop setup time elapsed. |  |  |  |  |
| done_tbuf | Out | 1 | Signal indicating I2C TBUF time elasped |  |  |  |  |
| done_sda_su | Out | 1 | Sinal indicating SDA setup time elapsed. |  |  |  |  |
| Signal Name | In/
Out | Width (bits) | Description |  |  |  |  |
| clk | In | 1 | “core_clk” connected if parameter SINGLE_CLK_DOMAIN = 0
“pclk” connected if parameter SINGLE_CLK_DOMAIN = 1 |  |  |  |  |
| rst_n | In | 1 | Asynchronous reset with rising edge synchronized to appropriate  clock input to the logic. Use “i2c_ctrl_rst_n” as described in Section 6.1 |  |  |  |  |
| i2c_en | In | 1 | As described in Table 18. |  |  |  |  |
| filter_en | In | 1 | As described in Table 18. |  |  |  |  |
| spike_pw_reg[3:0] | In | 4 | As described in Table 18. |  |  |  |  |
| hs_spike_pw_reg[3:0] | In | 4 | As described in Table 18. |  |  |  |  |
| hs_xfer_enabled | In | 1 | As described in Table 6. |  |  |  |  |
| tgt_filter_en_for_hs | In | 1 | As described in Table 12. |  |  |  |  |
| filter_in | In | 1 | Asynchronous input signal.
For SDA instance, SDA Input Buffer output should be connected to this port.
For SCL instance, SCL Input Buffer output should be connected to this port. |  |  |  |  |
| filter_out | Out | 1 | Synchronized output. Spikes will be filtered only when “i2c_en” and “filter_en” both are 1’b1. 
For SDA instance, this port should be connected to “sda_in” port of Receive Module
For SCL instance, this port should be connected to “scl_in” port of Receive Module |  |  |  |  |
| Signal Name | In/
Out | Width (bits) | Description |  |  |  |  |
| clk | In | 1 | Host Clock. This port is driven by core clock if the SINGLE_CLK_DOMAIN parameter is 0 else driven by host clock. |  |  |  |  |
| rst_ n | In | 1 | Asynchronous active low reset (synchronous to “clk”) |  |  |  |  |
| addr | In | AW | Request address from host bridge |  |  |  |  |
| wdata | In | DW | Request write data from host bridge |  |  |  |  |
| wbe | In | BEW | Request write data byte enable from host bridge |  |  |  |  |
| wr_req | In | 1 | Request write enable from host bridge |  |  |  |  |
| rd_req | In | 1 | Request read enable from host bridge |  |  |  |  |
| rdata | Out | DW | Response read data to host bridge |  |  |  |  |
| rd_vld | Out | 1 | Response read data valid to host bridge |  |  |  |  |
| hw_intr_sts | In | DW | Interrupt status input to interrupt module. All these inputs are synchronized to input “clk”. Any source interrupt generated in different clock domain is synchronized outside this module. |  |  |  |  |
| Intr | Out | 1 | Interrupt output. This interrupt will be edge type if the module parameter “EDGE_INTR” is 1 else the interrupt will be a level type. |  |  |  |  |
| i2c_en | Out | 1 | Control signal enabling the functionality of the IP. This signal should be synchronized to “core_clk” only if the parameter SINGLE_CLK_DOMAIN is 0. |  |  |  |  |
| filter_en | Out | 1 | Control signal to enable the Spike Filter functionality. This signal is valid only when i2c_en is 1’b1 |  |  |  |  |
| ip_mode | Out | 1 | This signal is synchronous to “pclk”.
Configuration Register Signal indicating Address Mode supported.
1 := Controller Mode
0 := Target Mode |  |  |  |  |
| ip_addr_mode | Out | 1 | This signal is synchronous to “pclk”.
Configuration Register Signal indicating Address Mode supported.
1 := 10-bit Address supported
0 := Only 7bit Address supported |  |  |  |  |
| en_clk_stretch | Out | 1 | This signal is synchronous to “pclk”.
Configuration Register signal to enable Clock Stretching logic. |  |  |  |  |
| target_addr | Out | 10 | This signal indicates the address of the IP when configured in Target Mode. Bits[6:0] are valid if 7bit address is supported. |  |  |  |  |
| hw_bytes_not_sent | In | 8 | As described in Table 3. |  |  |  |  |
| hw_target_addr_sts | In | 10 | As described in Table 3. |  |  |  |  |
| sta_hd_reg | Out | 16 | Timing register for START Hold |  |  |  |  |
| sta_su_reg | Out | 16 | Timing register for START Setup |  |  |  |  |
| sda_hd_reg | Out | 16 | Timing register for SDA Hold |  |  |  |  |
| sda_su_reg | Out | 16 | Timing register for SDA Setup |  |  |  |  |
| scl_low_reg | Out | 16 | Timing register for SCL Low cycle |  |  |  |  |
| scl_hi_reg | Out | 16 | Timing register for SCL High cycle |  |  |  |  |
| sto_su_reg | Out | 16 | Timing register for STOP Setup |  |  |  |  |
| tbuf_reg | Out | 16 | Timing register for TBUF |  |  |  |  |
| hs_sta_hd_reg | Out | 16 | Timing register for START Hold for High Speed Mode |  |  |  |  |
| hs_sta_su_reg | Out | 16 | Timing register for START Setup for High Speed Mode |  |  |  |  |
| hs_sda_hd_reg | Out | 16 | Timing register for SDA Hold for High Speed Mode |  |  |  |  |
| hs_sda_su_reg | Out | 16 | Timing register for SDA Setup for High Speed Mode |  |  |  |  |
| hs_scl_low_reg | Out | 16 | Timing register for SCL Low cycle for High Speed Mode |  |  |  |  |
| hs_scl_hi_reg | Out | 16 | Timing register for SCL High cycle for High Speed Mode |  |  |  |  |
| hs_sto_su_reg | Out | 16 | Timing register for STOP Setup for High Speed Mode |  |  |  |  |
| hs_mode_ctrl_addr_reg | Out | 3 | Controller ID sent in High Speed Command by Controller |  |  |  |  |
| hs_mode_en | Out | 1 | This signal validates all the high speed timing and configuration registers. This signal is double synchronized version of the configuration register “hs_mode_en” if the “pclk” is asynchronous to “core_clk” |  |  |  |  |
| request_device_id_reg | Out | 1 | This signal is synchronized version of “request_device_id_reg” if “pclk” is asynchronous to “core_clk”. If the clocks are synchronous then the “request_device_id_reg” can directly drive this signal |  |  |  |  |
| tgt_addr_is_10bit | Out | 1 | Configuration register indicating Target address is 10bit if the value is 1’b1 else it is 7bit |  |  |  |  |
| tgt_addr_reg[9:0] | Out | 10 | Target Device address whose Device ID needs to be read |  |  |  |  |
| send_start_byte | Out | 1 | Configuration register (send_start_byte) indicating Controller to transmit Start Byte. |  |  |  |  |
|  |  |  |  |  |  |  |  |
| gen_call_addr_en | Out | 1 | Configuration register indicating support for General Call Address request by Target |  |  |  |  |
| multi_ctrl_en | Out | 1 | Configuration register indicating Mutiple Controllers are present on the I2C Bus. This will enable the Clock Synchronization and Arbitration logic of the I2C IP Controller. |  |  |  |  |
| hw_revision[2:0] | In | 3 | Device ID response received by the Controller. Revision field |  |  |  |  |
| hw_part_id[8:0] | In | 9 | Device ID response received by the Controller. Part ID field |  |  |  |  |
| hw_manufacturer | In | 12 | Device ID response received by the Controller. Manufacturer ID field |  |  |  |  |
| tgt_revision[2:0] | Out | 3 | This field is valid only when IP is configured as Target. Target has to send this field in response to Device ID request received. |  |  |  |  |
| tgt_part_id[8:0] | Out | 9 | This field is valid only when IP is configured as Target. Target has to send this field in response to Device ID request received. |  |  |  |  |
| tgt_manufacturer | Out | 12 | This field is valid only when IP is configured as Target. Target has to send this field in response to Device ID request received. |  |  |  |  |
| tgt_dev_id_valid | Out | 1 | This signal validates the value on bus “tgt_dev_id_reg”. |  |  |  |  |
| hw_hs_cmd_ctrl_id_rcvd | In | 3 | As described in Table 12. |  |  |  |  |
| spike_pw_reg[3:0] | Out | 3 | Spike Pulse Width configuration register used for Standard/Fast/Fast mode |  |  |  |  |
| hs_spike_pw_reg[3:0] | Out | 3 | Spike Pulse Width configuration register used for High speed mode |  |  |  |  |
| hw_xmt_ctrl_state[3:0] | In | 4 | The current state of FSM of Transmit control module |  |  |  |  |
| hw_xmt_state[3:0] | In | 4 | The current state of FSM of Transmit module |  |  |  |  |
| hw_rcv_state[2:0] | In | 3 | The current state of FSM of Receive module |  |  |  |  |
| hw_rcv_ctrl_state[3:0] | In | 4 | The current state of FSM of Receive control module |  |  |  |  |
| hw_bus_not_idle | In | 1 | As described in Table 9. |  |  |  |  |
| hw_request_device_id_reg_clr | In | 1 | Signal from hardware logic to clear the Device ID request configuraiton. Used by Controller. |  |  |  |  |
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |
| hw_hs_cmd_ctrl_id_rcvd | In | 3 | Control ID received in High speed command by Target |  |  |  |  |
| hw_rxff_nempty_entries | In | 16 | Receive FIFO fill status |  |  |  |  |
| hw_txff_nempty_entries | In | 16 | Transmit FIFO fill status |  |  |  |  |
| soft_rst_reg | Out | 1 | Soft reset signal |  |  |  |  |
| rxff_dma_threshold_reg | Out | 2 | Receive FIFO DMA threshold configuration register |  |  |  |  |
| txff_dma_threshold_reg | Out | 2 | Transmit FIFO DMA threshold configuration register |  |  |  |  |
| csr_ff_wdata | Out | DW | Write data to Transmit FIFO |  |  |  |  |
| csr_ff_wbe | Out | BEW | Write data byte enable to Transmit FIFO |  |  |  |  |
| csr_ff_wr | Out | 1 | Write enable to Transmit FIFO |  |  |  |  |
| csr_ff_rd | Out | 1 | Read enable to Receive FIFO |  |  |  |  |
| csr_ff_rdata | In | DW | Read output from Receive FIFO |  |  |  |  |
| csr_ff_rvld | In | 1 | Read data valid from Receive FIFO |  |  |  |  |
| csr_sts_rd | Out | 1 | Signal indicating read enable to Status Register |  |  |  |  |
| csr_sts_vld | In | 1 | This signal is synchronized version of “csr_sts_rd” in Core clock domain if Host and Core clocks are different. All the status signals generated in Core clock domain hold value when read is being performed. This logic is external to CSR module. |  |  |  |  |
| Signal Name | In/
Out | Width (bits) | Description |  |  |  |  |
| wr_clk | In | 1 | Write Clock port |  |  |  |  |
| wr_rst_n | In | 1 | Asynchronous active low reset. Gets deasserted on rising edge of “wr_clk” |  |  |  |  |
| wr_clr | In | 1 | Signal to reset the FIFO pointers in “wr_clk” domain. |  |  |  |  |
| wr_req | In | 1 | Write enable port operating in “wr_clk” domain. |  |  |  |  |
| rd_clk | In | 1 | Read Clock port |  |  |  |  |
| rd_rst_n | In | 1 | Asynchronous active low reset. Gets deasserted on rising edge of “rd_clk” |  |  |  |  |
| rd_clr | In | 1 | Signal to reset the FIFO pointers in “rd_clk” domain. |  |  |  |  |
| rd_req | In | 1 | Read enable port operating in “rd_clk” domain |  |  |  |  |
| wr_en | Out | 1 | Write enable to memory operating in “wr_clk” domain. This signal will gated when Overflow error occurs. |  |  |  |  |
| wr_addr | Out | MEM_ADDR_W | Write address to memory operating in “wr_clk” domain |  |  |  |  |
| wr_nempty_entries | Out | MEM_ADDR_W | Signal indicating number of datawords filled in the FIFO. This signals is operating in “wr_clk” domain |  |  |  |  |
| wr_full | Out | 1 | Signal indicating FIFO full status operating in “wr_clk” domain |  |  |  |  |
| wr_almost_full | Out | 1 | Signal indicating FIFO Almost full status operating in “wr_clk” domain. This signal will be asserted when number of empty entries in FIFO is less than or equal to THRESHOLD Parameter. |  |  |  |  |
| wr_overflow_err | Out | 1 | This signal will be 1’b1 whenever “wr_full” is 1’b1 and “wr_req” is 1. This is generated in “wr_clk” domain. |  |  |  |  |
| rd_en | Out | 1 | Read enable to memory operating in “rd_clk” domain. This signal will gated when Underflow error occurs. |  |  |  |  |
| rd_addr | Out | MEM_ADDR_W | Read address to memory operating in “rd_clk” domain |  |  |  |  |
| rd_nempty_entries | Out | MEM_ADDR_W | Signal indicating number of datawords filled in the FIFO. This signals is operating in “rd_clk” domain. |  |  |  |  |
| rd_empty | Out | 1 | Signal indicating FIFO empty status operating in “rd_clk” domain. |  |  |  |  |
| rd_almost_empty | Out | 1 | Signal indicating FIFO Almost empty status operating in “rd_clk” domain. This signal will be asserted when number of empty entries in FIFO is less than or equal to THRESHOLD Parameter. |  |  |  |  |
| rd_underflow_err | Out | 1 | This signal will be 1’b1 whenever “rd_empty” is 1’b1 and “rd_req” is 1. This is generated in “rd_clk” domain. |  |  |  |  |
| Signal Name | In/
Out | Width (bits) | Description |  |  |  |  |
| pclk | In | 1 | APB Clock used by APB Logic |  |  |  |  |
| presetn | In | 1 | Asynchronous Active Low Reset. De-assertion of this reset  is synchronous to “pclk” |  |  |  |  |
| txff_dma_req | Out | 1 | DMA Request for Transmit FIFO |  |  |  |  |
| txff_dma_req_intr | Out | 1 | Rising edge of DMA Request for Transmit FIFO is used for interrupt |  |  |  |  |
| txff_dma_clr | In | 1 | DMA Clear for Transmit FIFO. Synchronize this signal before use. |  |  |  |  |
| rxff_dma_req | Out | 1 | DMA Request for Receive FIFO |  |  |  |  |
| rxff_dma_req_intr | Out | 1 | Rising edge of DMA Request for Receive FIFO is used for interrupt |  |  |  |  |
| rxff_dma_clr | In | 1 | DMA Clear for Receive FIFO. Synchronize this signal before use. |  |  |  |  |
| rxff_threshold_reg | In | 2 | As per Section 3.10. |  |  |  |  |
| txff_threshold_reg | In | 2 | As per Section 3.10. |  |  |  |  |
| txff_nempty_entries | In | Parameter
(AW) | Number of Non-empty entries in Transmit FIFO. 
This signal is synchronous to “pclk”. |  |  |  |  |
| rxff_rd_use_dw | In | Parameter
(AW) | Number of Non-empty entries in Receive FIFO.
This signal is synchronous to “pclk”. |  |  |  |  |
| Signal Name | In/
Out | Width (bits) | Description |  |  |  |  |
| pclk | In | 1 | APB Clock used by APB Logic |  |  |  |  |
| presetn | In | 1 | Asynchronous Active Low Reset. De-assertion of this reset  is synchronous to “pclk” |  |  |  |  |
| paddr | In | 12 | APB Address bus |  |  |  |  |
| psel | In | 1 | APB peripheral select signal |  |  |  |  |
| penable | In | 1 | APB Enable signal |  |  |  |  |
| pwrite | In | 1 | APB Write signal. Access is write when this signal is 1’b1 and Read when this signal is 1’b0. |  |  |  |  |
| pwdata | In | 32 | APB Write data |  |  |  |  |
| pstrb | In | 4 | APB Write strobe |  |  |  |  |
| pready | Out | 1 | APB Ready signal from the peripheral |  |  |  |  |
| prdata | Out | 32 | APB Read data from the peripheral |  |  |  |  |
| pslverr | Out | 1 | APB Slave Error signal |  |  |  |  |
| reg_addr | Out | 12 | Request Address. This signal is generated by the Address decoder part of the Slave Interface module. This signal is synchronous to “pclk”. This signal is sent to CSR module. |  |  |  |  |
| reg_wdata | Out | 32 | Write Data. This signal is synchronous to “pclk”. This signal is sent to CSR module. |  |  |  |  |
| reg_wbe | Out | 4 | Write Data Byte enable. This signal is synchronous to “pclk”. |  |  |  |  |
| reg_wr_req | Out | 1 | Write Enable to CSR Register generated by the Address decoder module of Slave Interface module. This signal is synchronous to “pclk” |  |  |  |  |
| reg_rd_req | Out | 1 | Read Enable to CSR Register generated by the Address decoder module of Slave Interface module. This signal is synchronous to “pclk” |  |  |  |  |
| reg_rdata | In | 32 | Read data from the CSR Module in sync with “pclk” |  |  |  |  |
| reg_rd_vld | In | 1 | Read Valid signal validating the “reg_rdata”. This signal is high for 1 “pclk”. |  |  |  |  |

### Cross-links
- [Receive Module](#intermediate)
- [Timing Generator Module](#intermediate)
- [CSR Module](#intermediate)
