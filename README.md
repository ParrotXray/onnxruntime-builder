# OnnxRuntime Builder

Automatically builds [ONNX Runtime](https://github.com/microsoft/onnxruntime) shared libraries for multiple targets and publishes them as GitHub Releases.

## Targets

| Name | Runner | AVX | AVX2 | AVX512 | Use Case |
|------|--------|-----|------|--------|----------|
| `linux-amd64-all` | ubuntu-24.04 | ON | ON | ON | Server-grade Intel/AMD (Skylake-X+) |
| `linux-amd64-avx2` | ubuntu-24.04 | ON | ON | OFF | Modern Intel/AMD (Haswell 2013+) |
| `linux-amd64-noavx2` | ubuntu-24.04 | OFF | OFF | OFF | Legacy x86_64 CPUs |
| `linux-aarch64` | ubuntu-24.04-arm | OFF | OFF | OFF | ARM64 (Raspberry Pi 4/5, AWS Graviton, etc.) |

## Triggers

- **Push**: triggers when `.github/workflows/build-onnxruntime.yml` is modified
- **Schedule**: runs daily at UTC 06:00, skips if upstream has no new release
- **Manual**: can be triggered from the Actions tab via `workflow_dispatch`

## How It Works

1. Fetch the latest release tag from `microsoft/onnxruntime`
2. Compare with `.last-built-version` to skip redundant builds
3. Build all 4 targets in parallel using GitHub-hosted runners
4. Package each target as a `.tar.gz` and publish as a GitHub Release
5. Commit the new version to `.last-built-version`

## Release Contents

Each `.tar.gz` contains:

```
libonnxruntime.so
libonnxruntime.so.<version>
include/
  onnxruntime_c_api.h
  onnxruntime_cxx_api.h
  onnxruntime_cxx_inline.h
  ...
```

## Usage

Download the appropriate release asset for your target machine:

```bash
# Example: Linux x86_64 without AVX2
wget https://github.com/<your-username>/onnxruntime-builder/releases/download/<version>-custom/linux-amd64-noavx2-<version>.tar.gz
tar -xzf linux-amd64-noavx2-<version>.tar.gz
```

Then link against `libonnxruntime.so` in your project.

## Notes

- AVX512 builds require a CPU that supports AVX-512 (Intel Skylake-X or later). Running an AVX512 binary on an unsupported CPU will cause `SIGILL`.
- ARM64 builds use NEON instead of AVX. The AVX flags are irrelevant on ARM and are set to OFF.
- GitHub Actions `schedule` triggers are automatically disabled after 60 days of repository inactivity. Manually trigger a run or push a commit to keep the workflow alive.
