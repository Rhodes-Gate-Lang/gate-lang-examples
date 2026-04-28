# ALU8 Gate-Lang Example

This directory contains a compositional 8-bit ALU built in Gate-Lang.

## Top-level component

- `ALU8(a:8, b:8, opcode:3, carry_in:1) -> result:8, carry:1, zero:1, negative:1, overflow:1`

## Opcode map

- `000` - `ADD`   (`a + b`)
- `001` - `SUB`   (`a - b`)
- `010` - `INC`   (`a + 1`)
- `011` - `DEC`   (`a - 1`)
- `100` - `NEG`   (`-a`)
- `101` - `ADC`   (`a + b + carry_in`)
- `110` - `SBC`   (`a - b - (1 - carry_in)`; implemented as `a + ~b + carry_in`)
- `111` - `PASS_A` (result is `a`)

## Flag behavior

- `carry`: final carry-out from the selected arithmetic datapath
- `zero`: `1` when `result == 0`
- `negative`: MSB/sign bit of `result`
- `overflow`: signed overflow for arithmetic ops, `0` for `PASS_A`

## Module layout

- `primitives.gate`: `Mux1`, `Mux8`, `Not8`, `IsZero8`
- `adders.gate`: `HalfAdder`, `FullAdder`, `RCA8`
- `arith.gate`: `Add8`, `Sub8`, `Inc8`, `Dec8`, `Neg8`, `Adc8`, `Sbc8`
- `alu8.gate`: top-level ALU wiring and flag selection

## Build and quick verification

From the repo root, using your aliases:

```sh
gatec gatec/examples/ALU/alu8.gate /tmp/gate-alu-out
```

Run vectors:

```sh
# ADD: 5 + 3 = 8
gate sim /tmp/gate-alu-out/ALU8.gateo --output-format bin \
  a=0b00000101 b=0b00000011 opcode=0b000 carry_in=0b0

# SUB: 3 - 5 = -2 (0xFE)
gate sim /tmp/gate-alu-out/ALU8.gateo --output-format bin \
  a=0b00000011 b=0b00000101 opcode=0b001 carry_in=0b1

# Overflow ADD: 0x7F + 0x01 -> 0x80, overflow=1
gate sim /tmp/gate-alu-out/ALU8.gateo --output-format bin \
  a=0b01111111 b=0b00000001 opcode=0b000 carry_in=0b0

# NEG edge: -0x80 -> 0x80, overflow=1
gate sim /tmp/gate-alu-out/ALU8.gateo --output-format bin \
  a=0b10000000 b=0b00000000 opcode=0b100 carry_in=0b0

# PASS_A: result follows a
gate sim /tmp/gate-alu-out/ALU8.gateo --output-format bin \
  a=0b10101010 b=0b11110000 opcode=0b111 carry_in=0b0
```
