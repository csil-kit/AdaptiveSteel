# AdaptiveSteel

AdaptiveSteel is an experimental OpenSees uniaxial material for cyclic steel response with strength, post-capping, and stiffness deterioration. An optional adaptive mode updates the response using the axial force of a monitored OpenSees element.

This repository makes AdaptiveSteel publicly available to support research
transparency and reproducibility. Its use, modification, and redistribution are
governed by the [AdaptiveSteel Research-Only License](LICENSE.md).

The author plans to propose AdaptiveSteel for inclusion as a built-in material
in a future OpenSees release. Inclusion and release timing are subject to review
and acceptance by the OpenSees maintainers.

## Using AdaptiveSteel in OpenSees

Install the library as described for your platform below. OpenSees constructs the material with the standard `uniaxialMaterial` command.

### Symmetric response

Use one set of backbone parameters in both loading directions:

```tcl
uniaxialMaterial AdaptiveSteel $matTag $Ke $dp $dpc $du $Fy $FmaxFy $FresFy $Lamda_S $Lamda_C $Lamda_K $c_S $c_C $c_K $D_pos $D_neg
```

To enable axial-load adaptation, append the monitored element and section properties:

```tcl
uniaxialMaterial AdaptiveSteel $matTag $Ke $dp $dpc $du $Fy $FmaxFy $FresFy $Lamda_S $Lamda_C $Lamda_K $c_S $c_C $c_K $D_pos $D_neg $eleTag $Pye $h_tw $Lb_ry
```

### Asymmetric response

Use separate positive- and negative-direction backbone parameters:

```tcl
uniaxialMaterial AdaptiveSteel $matTag $Ke $dp_pos $dpc_pos $du_pos $Fy_pos $FmaxFy_pos $FresFy_pos $dp_neg $dpc_neg $du_neg $Fy_neg $FmaxFy_neg $FresFy_neg $Lamda_S $Lamda_C $Lamda_K $c_S $c_C $c_K $D_pos $D_neg
```

The asymmetric form also supports axial-load adaptation:

```tcl
uniaxialMaterial AdaptiveSteel $matTag $Ke $dp_pos $dpc_pos $du_pos $Fy_pos $FmaxFy_pos $FresFy_pos $dp_neg $dpc_neg $du_neg $Fy_neg $FmaxFy_neg $FresFy_neg $Lamda_S $Lamda_C $Lamda_K $c_S $c_C $c_K $D_pos $D_neg $eleTag $Pye $h_tw $Lb_ry
```

Positive magnitudes must be supplied for both the positive- and negative-direction capacity parameters. The `_pos` and `_neg` suffixes select the loading direction; they do not indicate the sign of the input value.

### Parameters

| Parameter | Description |
| --- | --- |
| `matTag` | Unique positive integer material tag. |
| `Ke` | Initial elastic stiffness. |
| `dp`, `dp_pos`, `dp_neg` | Pre-capping deformation capacity. |
| `dpc`, `dpc_pos`, `dpc_neg` | Post-capping deformation capacity. |
| `du`, `du_pos`, `du_neg` | Ultimate deformation capacity. |
| `Fy`, `Fy_pos`, `Fy_neg` | Yield strength. |
| `FmaxFy`, `FmaxFy_pos`, `FmaxFy_neg` | Maximum-to-yield strength ratio, `Fmax/Fy`. |
| `FresFy`, `FresFy_pos`, `FresFy_neg` | Residual-to-yield strength ratio, `Fres/Fy`. |
| `Lamda_S` | Cyclic deterioration parameter for yield-strength deterioration. |
| `Lamda_C` | Cyclic deterioration parameter for post-capping-stiffness deterioration. |
| `Lamda_K` | Cyclic deterioration parameter for unloading-stiffness deterioration. |
| `c_S`, `c_C`, `c_K` | Rates of yield-strength, post-capping-stiffness, and unloading-stiffness deterioration. |
| `D_pos`, `D_neg` | Rates of cyclic deterioration in the positive and negative loading directions. |
| `eleTag` | Positive integer tag of the element whose local axial force is monitored. |
| `Pye` | Axial yield capacity. |
| `h_tw` | Web slenderness ratio, `h/tw`. |
| `Lb_ry` | Unbraced-length ratio, `Lb/ry`. |

The shared argument names, and their ordering in the asymmetric form, follow the OpenSees
[`IMKBilin`](https://opensees.github.io/OpenSeesDocumentation/user/manual/material/uniaxialMaterials/IMKBilin.html) material.
See that documentation for additional explanation of the shared parameter
concepts and their theoretical references. AdaptiveSteel adds its own axial-load
adaptation and should not be assumed to be identical to `IMKBilin`.

AdaptiveSteel does not impose a unit system. Use one consistent system for stiffness, strength, force, and deformation. The four ratio inputs `FmaxFy`, `FresFy`, `h_tw`, and `Lb_ry` are dimensionless.

When the material represents a rotational spring, deformation quantities are rotations and strength quantities are moments. When it represents an axial spring, they are axial deformations and forces.

### Limitations

- AdaptiveSteel is experimental research software and is not certified for safety-critical, life-safety, regulatory, or professional design decisions. Independently verify all analysis results.
- This binary has been validated only for the configurations listed below.
- OpenSees database serialization and distributed-memory workflows requiring `sendSelf` and `recvSelf` are not supported by this release.

## Binary release

| Platform | Library | Status |
| --- | --- | --- |
| macOS | `AdaptiveSteel.dylib` | Available and validated |
<!-- | Ubuntu Linux | `AdaptiveSteel.so` | Not yet released |
| Windows | `AdaptiveSteel.dll` | Not yet released | -->

## macOS (`.dylib`)

### Validated configuration

- File: `AdaptiveSteel.dylib`
- Format: Mach-O 64-bit dynamic library
- Architecture: Apple Silicon (`arm64`)
- Minimum operating system: macOS 26.0
- Build SDK: macOS 26.5
- OpenSees plugin entry point: `OPS_AdaptiveSteel`
- Validated host: OpenSees 3.6.0 (`arm64`)

### Runtime dependencies

The macOS binary depends only on the following macOS system libraries:

```text
/usr/lib/libSystem.B.dylib
/usr/lib/libc++.1.dylib
```

No absolute user or build-directory path is embedded in the library identity. The library identifies itself by the relative name:

```text
AdaptiveSteel.dylib
```

### Compatibility requirements

The target computer must meet all of the following requirements:

1. It must be an Apple Silicon Mac capable of running `arm64` applications.
2. It must run macOS 26.0 or newer.
3. The OpenSees process must run natively as `arm64`.
4. OpenSees must provide a C++ ABI compatible with the OpenSees 3.6.0 build used to validate this library.
5. OpenSees must be able to locate `AdaptiveSteel.dylib` through its runtime.

The library is not compatible with an Intel-only Mac or an `x86_64` OpenSees process running through Rosetta.

### Installation

Place the library in the user's binary directory:

```sh
mkdir -p "$HOME/bin"
cp AdaptiveSteel.dylib "$HOME/bin/AdaptiveSteel.dylib"
chmod 755 "$HOME/bin/AdaptiveSteel.dylib"
```

## License

AdaptiveSteel is experimental software made publicly available for
non-commercial academic and scientific research under the
[AdaptiveSteel Research-Only License](LICENSE.md). The license permits internal
research modifications but does not permit redistribution or publication of the
Software or modified versions without prior written permission.
