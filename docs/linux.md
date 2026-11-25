# Linux / Proton

LinUilian follows VItoriaBr and keeps its executable name, `victory.exe`.
Build the Windows target and run it under Wine/Proton. The portable Linux
test job checks platform-independent crates; it does not provide a native
Linux game runtime.

## Runtime requirements

- A Windows x64 diagnostics build using the pinned Rust toolchain, an MSVC
  linker and Windows SDK (native Windows or a Linux cross-build).
- GE-Proton 11-14's Wine and DXGI, with native `d3d12,d3d12core,dxgi` overrides.
- Microsoft's DXC and DXIL validator. Set `VICTORY_DXC` and `VICTORY_DXIL` to
  their Windows paths. Wine's built-in `d3dcompiler_47` is tried first; the
  imported fallback handles rejected HLSL and missing root signatures.
- A separate prefix, cache and data directory. Keep supplied game files
  read-only and all generated data outside the checkout. Run offline.

For our NVIDIA test, vkd3d-proton was built at `cae8353ed7959c4` with
dxil-spirv's `legacy-struct-remove` at
`e084e35c4506210b44f6b7df3bbf31a1026545b8`. This fixes three isolated compiler
recursions observed in an earlier VIctory fork under Wine. It does not by
itself establish correct rendering or fix GPU memory faults.

`VICTORY_NO_WATCHDOG=1` opts into the imported, build/signature-checked
title-watchdog hook. Use an external timeout when enabling it. It does not
disable the GPU driver's watchdog or resolve a hung GPU submission.

## Local verification

Initial target: CachyOS, RTX 5090, NVIDIA 620.73.04, GE-Proton 11-14.
The unmodified VItoriaBr baseline and Linux-adapted diagnostics cross-builds pass.
All-target, all-feature Windows Clippy with warnings denied passes, as does the
asset-policy check (178 files). Nine Windows library suites under Wine report
**542 passed, 7 failed, 10 ignored**. Integration fixtures were not run.

The failures are three pipeline-library expectations, the shader stub digest
expectation, an unchanged descriptor search-cursor assertion, and two HLE host
API checks (`WSAConnectByList` and device/product information). The three imported
DXC tests, indirect clamp and depth-plane rendering tests pass.

The first offline title run rendered startup/setup screens and reached world
submissions, but its window remained black before an NVIDIA Xid 13/MMU fault.
It logged no shader stack overflow and was stopped after the GPU fault at about
145 seconds. This is a successful Linux boot, **not working gameplay**.

That run used the upstream NVIDIA preset plus `VICTORY_SHADER_LOOP_LIMIT=32`,
`VICTORY_NO_WATCHDOG=1` and `VKD3D_DISABLE_EXTENSIONS=VK_NV_raw_access_chains`.
The low loop limit is a boot diagnostic that can truncate shader work, not a
rendering-correctness fix. A relay-disabled follow-up continued for about 3.5 minutes without a recorded
GPU fault, but remained black. Its launcher and observer were externally
terminated (exit 143), leaving no normal run result; this was inconclusive.

The upstream README's Windows results are retained below our fork notice
for attribution and reference. No game data, generated shaders, caches or
runtime binaries are distributed by this fork.

## Series S experiment

The `experiment/series-s` branch includes
[fxceless PR #798](https://github.com/fxceless/victory-fork/pull/798), preserving
its authorship. It selects the title's own upscaler instead of the FSR 4 path.
The original executable is read-only; patches apply to a separately named
prepared copy after build-hash and byte-signature checks. Use
`run <executable> --execute --console series-s`; `VICTORY_TITLE_PATCHES=0`
disables the built-in patches. Keep the Series S shader cache separate.

This branch passes the diagnostics build, Windows all-target/all-feature Clippy,
asset policy, and all 29 game/loader library tests under Wine, including the two
new patch regressions. Both expected signatures matched in the local prepare
check. Linux gameplay validation is still pending.
