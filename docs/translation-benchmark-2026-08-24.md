# Three-way CPU translation benchmark — 2026-08-24

Ferrum's first sealed three-way CPU campaign is complete. One freestanding
Win64 executable ran through:

1. Ferrum's direct Darwin/FEX driver;
2. CrossOver Preview's ARM64 Wine host with FEX; and
3. the same CrossOver Preview build's x86-64 Wine host under Rosetta 2.

This is a CPU translation microbenchmark, not a whole-game FPS result. It does
not measure Metal, Direct3D, shader compilation, audio, input or full Wine API
compatibility.

## Test host and protocol

- Apple M5, 10 logical CPUs, `FEAT_AFP=1`
- macOS 26.5.1 (25F80)
- CrossOver Preview 27.0.0.40921 (`20260821`)
- six fresh processes per route, in a balanced route order
- 34 kernels per process, nine steady samples per kernel
- each displayed value is the median of six per-process medians
- 3 route probes plus 18 measured processes; 34/34 kernels executed everywhere
- zero feature skips, guest errors, timeouts, timing-floor failures, thermal/load
  failures or scoped process leftovers

The executable, source, route components, app signature, fresh bottles, process
architectures, raw logs and cleanup records were digest-bound before reporting.
The final report was independently reconstructed from the sealed evidence and
matched byte for byte.

`↑` means higher is better. `↓` means lower is better. The exact numeric winner
is bold. `≈` means its lead over the named runner-up is 1% or less and should be
treated as a practical tie until more title-level evidence exists.

## Final steady results

| Benchmark | Better | Ferrum | FEX | Rosetta | Winner |
|---|---:|---:|---:|---:|---|
| `int_add_dep` | ↓ ns/op | 0.238 | 0.241 | **0.238** | Rosetta ≈ Ferrum |
| `int_add_independent` | ↑ Mops/s | 10905.604 | **11456.623** | 10946.467 | FEX |
| `int_mul_dep` | ↓ ns/op | 0.711 | 0.721 | **0.709** | Rosetta ≈ Ferrum |
| `int_mul_independent` | ↑ Mops/s | 5540.048 | 5515.380 | **5550.808** | Rosetta ≈ Ferrum |
| `int_variable_div_dep` | ↓ ns/div | **2.780** | 2.797 | 2.840 | Ferrum ≈ FEX |
| `int_popcnt_independent` | ↑ Mpopcnt/s | 3944.102 | 4157.684 | **5260.763** | Rosetta |
| `int_bitops_dep` | ↓ ns/op | 0.239 | **0.239** | 0.300 | FEX ≈ Ferrum |
| `branch_predictable` | ↑ Mdecisions/s | 3066.903 | 3543.613 | **4074.506** | Rosetta |
| `branch_random` | ↑ Mdecisions/s | 940.158 | 871.124 | **954.450** | Rosetta |
| `cmov_mixed` | ↑ Mselect/s | 2026.986 | **2075.123** | 2043.471 | FEX |
| `call_direct` | ↓ ns/call | **0.965** | 1.123 | 0.975 | Ferrum |
| `call_indirect_register` | ↓ ns/call | **1.102** | 1.324 | 1.133 | Ferrum |
| `sse_f32_dep` | ↓ ns/fp-op | **0.642** | 0.643 | 0.646 | Ferrum ≈ FEX |
| `sse_f32_independent` | ↑ Mflop/s | **5700.676** | 5675.043 | 5686.139 | Ferrum ≈ Rosetta |
| `sse_f64_dep` | ↓ ns/fp-op | 0.644 | 0.642 | **0.642** | Rosetta ≈ FEX |
| `sse_f64_independent` | ↑ Mflop/s | **5689.853** | 5680.592 | 5672.521 | Ferrum ≈ FEX |
| `sse_packed_f32_dep` | ↓ ns/lane-op | **0.161** | 0.161 | 0.161 | Ferrum ≈ FEX |
| `sse_packed_f64_dep` | ↓ ns/lane-op | 0.324 | **0.319** | 0.321 | FEX ≈ Rosetta |
| `sse_paddd_dep` | ↓ ns/lane-op | 0.119 | 0.120 | **0.119** | Rosetta ≈ Ferrum |
| `sse_shuffle_dep` | ↓ ns/instr | **0.478** | 0.481 | 0.955 | Ferrum ≈ FEX |
| `sse_cvttsd2si_mixed` | ↓ ns/conv | 0.717 | 0.722 | **0.715** | Rosetta ≈ Ferrum |
| `sse_sqrtps_dep` | ↓ ns/lane-sqrt | 0.780 | **0.773** | 0.774 | FEX ≈ Rosetta |
| `sse_divps_dep` | ↓ ns/lane-div | 0.656 | **0.651** | 0.655 | FEX ≈ Rosetta |
| `sse_denormal_dep` | ↓ ns/fp-op | 0.639 | **0.636** | 0.642 | FEX ≈ Ferrum |
| `sse_dpps_dep` | ↓ ns/dot | **3.358** | 3.359 | 3.865 | Ferrum ≈ FEX |
| `avx2_paddd_dep` | ↓ ns/lane-op | 0.339 | **0.337** | 0.338 | FEX ≈ Rosetta |
| `fma_v8sf_dep` | ↓ ns/fp-op | 0.172 | 0.172 | **0.152** | Rosetta |
| `memory_sequential_read_4mib_working_set` | ↑ GB/s | 31.445 | **75.647** | 75.628 | FEX ≈ Rosetta |
| `memory_sequential_write_4mib_working_set` | ↑ GB/s | 25.823 | 52.190 | **52.256** | Rosetta ≈ FEX |
| `memory_pointer_chase_1mib_working_set` | ↓ ns/load | **3.518** | 3.609 | 4.100 | Ferrum |
| `memory_rep_movsb_4mib_working_set` | ↑ GB/s | **80.181** | 49.062 | 4.049 | Ferrum |
| `memory_hot_memcpy_256` | ↑ GB/s | 56.420 | 56.385 | **56.633** | Rosetta ≈ Ferrum |
| `atomic_xadd` | ↓ ns/atomic | **1.665** | 1.682 | 1.667 | Ferrum ≈ Rosetta |
| `atomic_cmpxchg` | ↓ ns/atomic | 1.651 | 1.656 | **1.538** | Rosetta |

## What the results say

The exact row-win count is Rosetta 13, Ferrum 12 and FEX 9, but that is not the
most useful summary: 23 of 34 top-two gaps are at or below 1%. Requiring a lead
larger than 1% leaves only five clear Rosetta wins, four Ferrum wins and two FEX
wins.

Using the same descriptive 0.8–1.25× parity band as earlier public comparisons:

| Candidate versus Rosetta | Within band | Slower than band | Faster than band |
|---|---:|---:|---:|
| Ferrum | 27 | 4 | 3 |
| FEX | 30 | 1 | 3 |

The largest repeatable differences are more informative than the win count:

- Ferrum and FEX complete `rep movsb` at 80.181 and 49.062 GB/s versus
  Rosetta's 4.049 GB/s: 19.80× and 12.12× Rosetta.
- Ferrum and FEX execute the dependent SSE shuffle in about 0.48 ns versus
  0.955 ns for Rosetta: roughly 2× Rosetta's speed.
- Rosetta leads POPCNT by 26.5% over FEX and the predictable branch test by
  15.0% over FEX.
- Rosetta leads FMA latency by 13.5% over FEX.
- Ferrum leads FEX on direct calls, indirect calls and pointer chasing.
- In the cache-sensitive 4 MiB working-set sequential read/write kernels, FEX
  and Rosetta match. Ferrum reaches only 31.445/25.823 GB/s versus about
  75.6/52.2 GB/s, a real route-specific anomaly now queued for investigation.
  These rows are not a DRAM-bandwidth claim.

Scalar SSE float/double, float-to-integer conversion and denormal rows are near
parity on this host. This Apple M5 has AFP hardware; these rows must not be
generalised to an AFP-less M1 Max.

Route-median variability above 5% remains common, especially on cold and memory
tests. The large gaps above exceed that noise, but close rows should be rerun on
more machines and real applications before product decisions depend on them.

## First optimization follow-up

Milestone m1050 folds a one-hot x86 `TEST` followed by `JZ`/`JNZ` to one native
AArch64 `TBZ`/`TBNZ` when every TEST flag is dead. On the exact
`branch_predictable` hot block, independent Apple disassembly confirms the host
shape changed from AND+CMP+B.eq to TBZ: 13 to 11 instructions and 8 bytes
smaller. The final generated-code matrix passed 15/15 on an Apple M2, including
six negative controls, and a freestanding semantic PE passed both baseline and
patched drivers.

This does not revise the table above. An earlier A/B of the same hot-block fold
measured patched/baseline throughput of 0.999016x, so no speed win is claimed.
The useful result is narrower generated code with exact flag-semantics guards;
further branch work remains open.

## Second optimization follow-up

The next review added a compile-only witness and balanced A/B runner so exact
x86 bytes can be tied to their FEX IR and Apple AArch64 code shape before a
performance change is accepted. The measured POPCNT block was already four
scalar CSSC `cnt` instructions on the Apple host, so no POPCNT production edit
was made.

A narrower FMA load-pair candidate was tested in three balanced A/B pairs (six
complete 34-kernel runs). Its candidate/baseline FMA ratios were `0.991624x`,
`0.991352x`, and `0.991830x`; the aggregate was `0.991624x`. No non-FMA
aggregate fell below `0.97x`, but the target row did not improve, so the
production change was rejected and reverted byte-for-byte. The result table is
unchanged. This is the intended acceptance rule: a plausible code-shape idea is
not a speed milestone until measurement says it is one.

## Launch diagnostic

Median host launch-to-first complete guest identity record was 27.774 ms for the
direct Ferrum driver, 984.924 ms for CrossOver/FEX and 3542.489 ms for
CrossOver/Rosetta. This is a translation-pipeline diagnostic, not an end-user app
startup comparison: Ferrum's route does not yet include the same full Wine launch
stack, and launch variability was high.

## Evidence identities

- campaign: `6b03db91cd82280285856642e6a7045d83914ed868af33217a83ff6cc3419686`
- result JSON: `2bc9ae22907bc317f0ae0361f740176d7a233ad4251004d8c759e31814db411e`
- result Markdown: `c2d2ae208484d92161ba0cfb74753a5eaf2ff141ae07d4e39be5dbce5de13818`
- benchmark source: `4c8611fe171ac5ae3a9930846a35aac088d1ea48da2f12ff1ad20adcea6ae8f7`
- campaign runner: `3fcea00a91b73c2026dabb97c68743bb56605ab6ec06ffb2fc2b61df34446856`

These hashes identify the private sealed evidence used for the public summary;
they are not a claim that private runtime code has been published.
