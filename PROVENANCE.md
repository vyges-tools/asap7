# Provenance of this mirror

This repository is a **composite mirror** of ASAP7 collateral, assembled from two upstream
repositories at pinned commits. It is not a fork of any single one, and it is not the ASAP7
distribution in full.

| Path | Upstream | Pinned commit |
| --- | --- | --- |
| `models/`, `calibre/`, `cdslib/`, `asap7ssc7p5t_05/`, `docs/` | [The-OpenROAD-Project/asap7_pdk_r1p7](https://github.com/The-OpenROAD-Project/asap7_pdk_r1p7) | `58d72c9d291e186a77468586ab0c43d8a21eda6a` (2024-07-18) |
| `lef/`, `gds/`, `cdl/` | [The-OpenROAD-Project/asap7sc7p5t_28](https://github.com/The-OpenROAD-Project/asap7sc7p5t_28) | `875fd1eee58741378d875d9b81c95526a9b8c47c` (2026-04-11) |

## A correction worth recording

The descriptor previously named `The-OpenROAD-Project/asap7` as this mirror's origin. That was
wrong twice over: that repository is a **superproject of five submodules** and holds none of the
content itself, and what is actually here came from `asap7_pdk_r1p7` — one of those submodules.
A non-recursive clone of a superproject is how a mirror ends up looking complete while carrying
none of the collateral, which is exactly what happened.

That is also why there was no LEF or GDS: they live in the *standard-cell* repositories
(`asap7sc7p5t_27`, `asap7sc7p5t_28`), not in the process PDK repository, so no amount of
re-mirroring `asap7_pdk_r1p7` would ever have produced them.

## What is here, and what is not

Only a subset is mirrored. Upstream `asap7sc7p5t_28` is ~2.9 GB and its Liberty alone is ~74 MB
compressed (~500 MB expanded); this mirror is ~24 MB.

| | mirrored |
| --- | --- |
| tech LEF (1x and 4x) | yes |
| 4x cell LEF, GDS, LVS CDL — R, L, SL, SRAM | yes |
| Liberty NLDM — **RVT only**, TT/SS/FF, SIMPLE + SEQ + INVBUF | yes, decompressed from upstream `.lib.7z` |
| Liberty for LVT / SLVT / SRAM, CCS, AO/OA groups | **no** — take from upstream |

**One cell library is declared, not four.** `asap7sc7p5t_28_rvt` is the only entry in
`cell_libraries`, and every path it claims — LEF, GDS, CDL and Liberty for all three corners — is
present. The L, SL and SRAM physical collateral *is* mirrored and is listed under
`collateral.additional_cell_*`, but declaring them as cell libraries without their timing would
repeat exactly the defect this change exists to fix: a descriptor naming files that are not there.

The previous descriptor declared two cell libraries and named ten `lib/*.lib` paths, none of which
existed — upstream does not ship uncompressed Liberty at all.

## License and attribution

ASAP7 is **BSD 3-Clause**, Copyright 2022 Lawrence T. Clark, Vinay Vashishtha, or Arizona State
University. Redistribution requires the copyright notice and conditions to be retained; the
upstream notices are kept at [`LICENSE`](LICENSE) and
[`lef/LICENSE.asap7sc7p5t_28`](lef/LICENSE.asap7sc7p5t_28).

The Liberty files here were **decompressed** from upstream `.lib.7z`; their content is unmodified
and each retains its own BSD notice in the file header.

ASAP7 is a **predictive** 7 nm PDK for research and education. It is not a foundry process and
nothing here can be fabricated.

## Verifying this mirror

Every path the descriptor claims should exist. The shipped tooling checks it directly:

```sh
ASAP7_PDK=$PWD vyges pdk-store verify asap7.vyges-pdk.json
# verify OK: 19 collateral path(s) present
```

Or without the CLI:

```sh
python3 - <<'CHECK'
import json, pathlib
d = json.load(open('asap7.vyges-pdk.json'))
paths  = [v for v in d['collateral'].values() if isinstance(v, str)]
paths += [x for v in d['collateral'].values() if isinstance(v, list) for x in v]
paths += [c[k] for c in d['cell_libraries'] for k in ('lef', 'gds', 'spice') if k in c]
paths += [l for c in d['cell_libraries'] for ls in c.get('lib', {}).values() for l in ls]
paths += [m for c in d['corners'] for m in c.get('models', [])]
bad = [p for p in paths if not p.startswith('$') and not pathlib.Path(p).exists()]
print(f"{len(paths)-len(bad)}/{len(paths)} present" + ("" if not bad else f" - MISSING: {bad}"))
CHECK
```
