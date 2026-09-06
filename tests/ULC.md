# ULC output saturation regression

Build against the installed SDK **after rebuilding and installing libdragon**:

```sh
make -C tests testrom_ulc.z64
```

Run `tests/testrom_ulc.z64` in an accurate N64 emulator (or on a separately
authorized test device). The console and IS Viewer report
`ULC_TEST_PASS cases=260 failures=0` on success. `ULC_TEST_FAIL`, an assertion,
or failure to reach the terminal marker is a failed run. The test needs no
filesystem assets and does not write saves or use commercial audio.

The test drives the installed `rsp_ulc` overlay directly. It compares mono
synthesis against signed-saturating doubling of identical unscaled synthesis,
without float-reference rounding, codec noise-fill, mixer resampling, or AAC.
Cases cover:

- Positive/negative peaks, exact signed boundaries, and quiet samples.
- Direct full-block prefix, windowed, and mixed overlap regions.
- 128/256/512-sample subblocks, including both reshuffle emission sources.
- Unchanged retained overlap state and no writes past the output range.
- Discarded/preroll output and repeated commands.
- Stereo mid/side reconstruction across all pairs of the boundary values.

On the pre-fix SDK revision `1617eca8e703b0eedd5d657cf3b868b91d3e277f`, the
same test fails 72 of 260 cases: all 64 loud mono cases plus eight quiet mixed/
windowed cases affected by carry left by wrapping additions. The fixed overlay
passes all 260 in Gopher64. Its text is 4096 bytes, within the RSP IMEM limit;
the linker also enforces this limit.

This regression is an RSP synthesis test, not an end-to-end encoder, seeking,
streaming, or hardware qualification. Those require additional integration tests.
