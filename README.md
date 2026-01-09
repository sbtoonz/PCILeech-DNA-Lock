# DNA Locker for Xilinx Artix-7

## What is this?

dnalocker is a small Verilog module that locks your FPGA firmware to a specific Xilinx Artix-7 device using the chip’s built-in unique DNA. The protected logic will only run if the FPGA DNA matches the value you provide.

This is useful to:

- Prevent firmware reuse on unauthorized boards

- Discourage casual cloning of bitstreams

- Tie a design to a specific physical device

How it works

- Uses the Xilinx DNA_PORT primitive to read the 57-bit device DNA

- Shifts out the DNA bits at startup

- Compares them against a hard-coded expected value

- Produces a match signal

If match == 1, the board is authorized and your logic is enabled.
If match == 0, protected logic stays disabled.

Files

- dna_check.sv - the DNA comparison module

- Your top-level or FIFO module - instantiates dna_check and gates logic with dna_match

Quick start

1. Add dna_check.sv to your project

Include the file in your Vivado sources.

2. Instantiate in your fifo module just before the tick64
```
wire dna_match;
wire [56:0] expected_dna = 57'h0DEADBEEFCAFE; // Put your DMA cards DNA ID here as hex

dna_check dna_check_inst (
    .clk(clk),
    .expected_dna(expected_dna),
    .match(dna_match)
);
```
Replace 57'h0DEADBEEFCAFE with your real board DNA for production.

3. Lock your logic

Example: lock outgoing TLP packets in pcileech_fifo.

Original:
```
assign dtlp.tx_valid = dcom.com_dout_valid & `CHECK_MAGIC & `CHECK_TYPE_TLP;


        .p3_wr_en       ( dtlp.rx_valid[0]),
        .p4_wr_en       ( dtlp.rx_valid[1]),
        .p5_wr_en       ( dtlp.rx_valid[2]),
        .p6_wr_en       ( dtlp.rx_valid[3]),
```

Locked:
```
assign dtlp.tx_valid = dna_match & dcom.com_dout_valid & `CHECK_MAGIC & `CHECK_TYPE_TLP;

        .p3_wr_en       ( dtlp.rx_valid[0]  & dna_match ),
        .p4_wr_en       ( dtlp.rx_valid[1]  & dna_match ),
        .p5_wr_en       ( dtlp.rx_valid[2]  & dna_match ),
        .p6_wr_en       ( dtlp.rx_valid[3]  & dna_match ),

```
You can AND dna_match into any enable, valid, or state transition signal you want to protect.

Convert the result to a 57-bit hex value and paste into expected_dna.

Notes

* Only 57 bits of the 64-bit DNA are accessible in 7-series devices

* This does not replace bitstream encryption; it is a lightweight binding mechanism

License / Attribution

Inspired by Plasma2450 and Mark Harvey’s device-lock concepts.

Use at your own risk.

