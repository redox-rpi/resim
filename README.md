# resim (redox-rpi)

A simulator for the VideoCore IV VPU, the processor of the Raspberry Pi
that runs the boot firmware. redox-rpi uses its `vc4emul` to test vc4-tcc,
our C compiler for the VPU ([redox-rpi/tinycc](https://github.com/redox-rpi/tinycc),
`tests/vc4`), against host gcc and against vc4-elf-gcc. Versions and build
scripts for the whole toolchain: [redox-rpi/vc4-toolchain](https://github.com/redox-rpi/vc4-toolchain).

## Where it comes from

| Who | When | What |
|---|---|---|
| Mathias Gottschlag ([mgottschlag/resim](https://github.com/mgottschlag/resim), the original) | 2012 | libresim and the VideoCore IV scalar core, `vc4emul`; `LICENSE` (MIT) added 2020 |
| Julian Brown ([itszor/resim](https://github.com/itszor/resim)) | 2015-16 | mini-UART output, a DejaGNU target for the vc4 GCC test suite, more instructions and fixes; merged here |
| andryblack | 2022 | removed the Qt dependency; merged here (part of itszor's) |
| Michael Bishop ([cleverca22/resim](https://github.com/cleverca22/resim)) | 2020 | `switch`, `abs`, `mulhd.uu` and more for running the boot ROM; only these three taken here, reworked |
| redox-rpi | 2026 | the changes below |

A comparison of all forks is in rpi-open-firmware
[docs/forks.md](https://github.com/redox-rpi/rpi-open-firmware/blob/redox-rpi/docs/forks.md).

## Our changes

- `asr` was wrong for negative values and for a shift by 0 (found by
  vc4-tcc's csmith test).
- `bitflip`, `switch.b`, `switch` (jump tables), `abs` and `mulhd.uu`
  were missing. GCC code now runs without `-fno-jump-tables`.
- `VC4EMUL_COUNT=1` prints the executed instructions at exit;
  `VC4EMUL_PROF=file` writes a histogram of pc values.
- Builds with current g++ and CMake 4.

Known gaps: no `ldsb` (signed byte load), no vector (VRF) instructions,
floating point only partly; Julian Brown marked several of his
semantics as guessed.

## Build and run

```bash
cmake -S . -B build
make -C build vc4emul
build/vc4emul/vc4emul mem.bin 0 0     # flat image loaded at 0, start at 0
```

Programs print through the simulated mini UART. tinycc's `tests/vc4`
(`sim-crt.S`, `sim-rt.c`, `vc4-sim.ld`) shows how to build a program for it.

## Licence

MIT, see `LICENSE` (Mathias Gottschlag). The additions by Julian Brown
and Michael Bishop were published as public forks of this MIT project and
state no licence of their own; our changes are MIT.
