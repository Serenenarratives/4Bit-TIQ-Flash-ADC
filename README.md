# 4-Bit Flash ADC using CMOS Inverters as Comparators (TIQ)

A 4-bit flash Analog-to-Digital Converter designed in the **SkyWater SKY130 (130 nm)** open-source PDK. Instead of a resistor ladder with conventional comparators, this design uses the **Threshold Inverter Quantizer (TIQ)** technique: each comparator is a pair of cascaded CMOS inverters whose switching threshold (Vm) is set by transistor sizing. A **clocked transmission gate** at each comparator output samples the result before it is passed to the encoder.

## Features

- 4-bit resolution (15 comparators, thermometer code to binary encoder)
- Comparator threshold set by W/L ratio of the inverter, no resistor ladder needed
- Clocked digital comparator (inverter, inverter, transmission gate) with CLK and CLK-bar
- Supply: 1.8 V, simulated input ramp 0 to 0.9 V
- Designed and simulated with open-source tools (Xschem + Ngspice)

## Working Principle

### TIQ comparator

Each comparator is two cascaded inverters (M1/M2 and M3/M4). The second stage must be identical to the first so both keep the same DC threshold and the rising/falling transitions stay balanced at high input frequency. The switching threshold of the inverter is approximately:

```
        sqrt(μp·Wp / μn·Wn) · (Vdd − |Vtp|) + Vtn
Vm = ------------------------------------------------
              1 + sqrt(μp·Wp / μn·Wn)
```

where Vtn and Vtp are the NMOS/PMOS threshold voltages, Kn = (W/L)n · μn · Cox and Kp = (W/L)p · μp · Cox.
By varying the W/L ratio of each inverter, 15 different Vm values are generated, one per quantization level.

### Clocked output stage

A transmission gate (driven by CLK and CLK-bar) sits at the comparator output. The comparator result is passed to the encoder only during the clock-high phase, giving a synchronous, latched thermometer code that is then converted to 4-bit binary.

```
Vin -> [INV M1/M2] -> [INV M3/M4] -> [Transmission Gate (CLK, CLK̅)] -> Vo -> Encoder -> b3 b2 b1 b0
```

## Repository Structure

```
.
├── 4bit-tiq.sch        # Xschem schematic (top level)
├── netlist/            # Generated SPICE netlist
├── results/            # Simulation screenshots (see below)
└── README.md
```

> Adjust the folder names above to match your repo.

## Tools

| Tool | Purpose |
|------|---------|
| Xschem | Schematic capture |
| Ngspice | SPICE simulation |
| SKY130 PDK | 130 nm process models |

## Simulation Setup

| Parameter | Value |
|-----------|-------|
| VDD | 1.8 V |
| Input (Vin) | Ramp, 0 to 900 mV over 50 ns |
| Clock | 0 to 1.8 V pulse, 10 ns period (100 MHz), 50% duty |
| Analysis | Transient (`tran`), 0 to 50 ns |

## Results

### DC operating point
At Vin = 0 V and VDD = 1.8 V, the first inverter outputs of the comparator chain sit near 1.8 V (~1.795 V) and the second-stage nodes near 0 V (µV to sub-mV range), confirming each comparator is correctly biased in its saturated state.

![DC operating point](results/dc_op_point.jpg)

### Input ramp
Linear input ramp from 0 to 900 mV over 50 ns.

![Input ramp](results/vin_ramp.jpg)

### Clock
Clock pulse with 10 ns period, 0 to 1.8 V. (The blue trace is the same clock offset by +2.2 V for visualization.)

![Clock](results/clk.jpg)

### Digital outputs (b0 to b3)
Output bits b0 to b3 plotted with vertical offsets (+0, +2, +4, +6 V) for readability. The outputs update in step with the 10 ns clock, showing the sampled behavior of the clocked comparators and encoder.

![Digital outputs](results/outputs.jpg)

## Observations / Notes

- Outputs change in sync with the clock edges as expected from the clocked comparator stage.
- Some glitching on the LSBs (b0, b1) near clock transitions is visible; this is typical of an encoder driven by comparators with slightly different delays.
- Accuracy depends on how well the inverter thresholds are spaced; process/temperature corner simulation is recommended as future work.

## Future Work

- Fine-tune W/L ratios for uniform LSB spacing
- Add INL/DNL analysis
- Corner (TT/FF/SS) and temperature sweeps
- Layout (Magic) and post-layout simulation
- Add a sample-and-hold at the input

## References

- Clocked Digital Comparator / TIQ-based flash ADC design (IEEE-style paper, Fig. 1 and Vm equation used as reference for this design)
- SkyWater SKY130 PDK: https://github.com/google/skywater-pdk

## Author

Your Name, Department of Electronics Engineering, Your College

## License

This project is released under the MIT License. See `LICENSE` for details.
