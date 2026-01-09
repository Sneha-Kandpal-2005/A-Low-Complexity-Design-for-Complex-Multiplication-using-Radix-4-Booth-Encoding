

# A Low-Complexity Design for Complex Multiplication using Radix-4 Booth Encoding

**Short description**

This repository contains a parametrized complex multiplier implemented in Verilog along with a hardware demo wrapper (`top_demo`) that uses Vivado's **VIO (Virtual Input/Output)** to interactively drive inputs and observe outputs on an FPGA. The project also includes synthesizable top modules for different widths, testbenches for simulation, utility modules (clock divider, CSAT, PPG generators, Kogge-Stone adder, etc.), and constraint files for multiple target widths.

---

## Table of contents
- Project overview
- Repository layout (files & hierarchy)
- Supported configurations / parameters
- Simulation (Vivado XSIM) — how to run
- Synthesis & Implementation (Vivado) — step-by-step
- Hardware demo using VIO — GUI instructions & recommended configuration
- Running on-board (programming bitstream + debug)
- Troubleshooting & tips
- Contributing & license

---

## Project overview

The core functionality implements the complex multiplication of two complex numbers:

*(a + j b) × (c + j d) = (ac − bd) + j(ad + bc)*

The design uses Booth encoders, PPG matrix generators and carry-save adder trees (CSAT) combined with a Kogge-Stone adder to produce `re_part` (real) and `im_part` (imag) outputs. The `top_demo` module wraps the synthesizable `top` module and connects it to a Vivado VIO instance so you can drive inputs (`a`, `b`, `c`, `d`) from Vivado's Hardware Manager and observe outputs and verification signals in real time.

---

## Repository layout (suggested)

```
/ (project root)
├─ README.md                
├─ src/
│  ├─ top_demo.v            <- VIO-based demo wrapper (top_demo)
│  ├─ top.v                 <- parametric synthesizable top module (M param)
│  ├─ clock_divider.v       <- clock divider used by top_demo
│  ├─ booth_encoder_ppg.v
│  ├─ ppg_matrix_generator.v
│  ├─ nppg_matrix_generator.v
│  ├─ csat_8.v
│  ├─ csat_16.v
│  ├─ csat_32.v
│  ├─ csat_64.v
│  ├─ kogge_stone_adder.v
│  └─ ...other modules
├─ sim/
│  ├─ tb_top.v              <- top testbench used for simulation
│  └─ csat_32_tb.v          <- other verification testbenches
├─ constraints/
│  ├─ hardware_constraints.xdc
│  ├─ 8_bit.xdc
│  ├─ 16_bit.xdc
│  └─ 32_bit.xdc

```




## Supported configurations / parameters

- The `top` module is parameterized by `M` (bit-width of real/imag inputs). Common supported values in this repo: **8, 16, 32, 64**. Some CSAT modules have specialized implementations (csat_8, csat_16, csat_32, csat_64) — `top` selects the appropriate one using `if (M==...)` branches.
- `top_demo` parameter: `M` (default can be set in your instantiation)

Notes:
- Output width for `re_part` and `im_part` is `[2*M:0]` (signed representation).
- Inputs in `top_demo` are driven by the VIO outputs as unsigned vectors but expected arithmetic uses signed interpretation; `top_demo` uses `wire signed` casts to compute the expected reference values.

---

## Simulation

### Using Vivado XSIM

1. Open Vivado and create a new project (or use the existing project). Add the `src/` and `sim/` files to the project.
2. Set the simulation top to `tb_top` (or `tb_top.v`) in the Simulation Sources.
3. Run behavioral simulation (Flow -> Run Simulation -> Run Behavioral Simulation).
4. Use the waveform viewer to inspect inputs/outputs. You can also add `expected_re`/`expected_im` signals from `top_demo` or testbench to validate logic.



---

## Synthesis & Implementation (Vivado)

### GUI steps (quick)
1. Create a Vivado project (RTL project), add files from `src/`, set top module to `top` (for synthesis) or `top_demo` (if you want to build a bitstream ready for the VIO hardware demo).
2. Add the required constraint file under `Constraints/` (match M width and pins): `8_bit.xdc`, `16_bit.xdc`, `32_bit.xdc`for computational calculations or the `hardware_constraints.xdc` for the hardware demo.
3. If you plan to use VIO: instantiate `vio_0` IP (for the hardware demo) or include the provided `vio_0.xci` in the `ip/` folder.
4. Run **Synthesis → Implementation → Generate Bitstream**.


## Hardware demo using VIO (Vivado Hardware Manager)

`top_demo.v` is provided specifically for hardware demos using VIO. It instantiates:
- `clock_divider` to down-convert the on-board clock to the VIO-friendly rate to avoid timing violations.
- `vio_0` (the VIO IP) — **you must create this IP in Vivado** and match probe widths listed below.

### Creating VIO IP (GUI)
1. In Vivado IP Catalog search for **VIO (Virtual Input/Output)** and add it.
2. Configure probes as:
   - **Output probes** (from VIO to hardware inputs): 4 probes of width `M` (for `a,b,c,d`).
   - **Input probes** (hardware to VIO): `re_part`, `im_part`, `expected_re`, `expected_im` each width `2*M+1` and a 3-bit probe for match flags.
3. Generate output products and add the `vio_0` component to your project block design or to the RTL project.
4. If not using block design, instantiate the `vio_0` module as shown in `top_demo.v`.

### In Hardware Manager
1. Program the FPGA with the generated bitstream.
2. Open Hardware Manager → Open Target → Auto Connect.
3. Launch the VIO console; you’ll see editable fields for `a,b,c,d` and monitor-only fields for `re_part, im_part, expected_re, expected_im, match`.
4. Change inputs live and observe outputs update in real-time.

---

## Running on-board (programming + debug)
1. Connect board and power it.
2. Program device with bitstream (Vivado → Program Device).
3. Open VIO core in Hardware Manager to drive inputs.


---

## Testbench & verification strategy
- `tb_top.v` (in `sim/`) is the recommended simulation top: it should apply a set of vectors (including corner cases: max positive, max negative, sign-crossing, zero) and assert that `re_part` / `im_part` equal the expected results.



## Troubleshooting & tips
- If Vivado complains about width mismatches for VIO probes, re-check the probe widths: output probes `M`, input probes `2*M+1`.
- If `top_demo` isn't updating in Hardware Manager, ensure the VIO IP is connected to the clock that is toggling (we use a `clock_divider` output `clk_out` in the provided wrapper).
- For timing violations: start with `M=8` or `M=16`, examine synthesis reports, and pipeline where necessary.
- If you see mismatch flags toggle on hardware, sample slower clock or use ILA to capture waveform and debug partial sums.

---
