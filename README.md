# Scroll rendering, ink inference and CT data fixes

November 2026 Vesuvius Challenge Progress Prizes · [mydd7](https://github.com/mydd7)

These patches address problems found while rendering and inspecting public PHerc0800 CT and preparing ink labels from PHerc. Paris 4. The before-and-after tests are linked below.

## Render and inspect a segment in Villa

| Problem | Result on PHerc0800 | Evidence |
| --- | --- | --- |
| The renderer wrote the outputs but hung at Windows DLL shutdown. A batch job could not move on to ink detection. | Three runs exited normally. All 30 output files matched the baseline by SHA-256. A second tester reported three clean exits on Windows. | [Villa #2566](https://github.com/ScrollPrize/villa/pull/2566) · [Original report](https://github.com/ScrollPrize/villa/issues/2562) · [Independent Windows test](https://github.com/ScrollPrize/villa/pull/2566#issuecomment-6980777080) |
| The ink map showed nonzero scores in pixels whose selected CT layers were all zero. | 310,848 nonzero predictions in empty areas went to zero. All 497,600 CT-supported output pixels were unchanged. | [Villa #2567](https://github.com/ScrollPrize/villa/pull/2567) · [Before and after](https://raw.githubusercontent.com/mydd7/villa/578429f4454433649bc2c52e49cc088d4a6d2139/ink-empty-pixels.png) · [Counts and hashes](https://raw.githubusercontent.com/mydd7/villa/578429f4454433649bc2c52e49cc088d4a6d2139/ink-empty-pixels.json) |

Both Villa examples use segment `20251029010146` from volume `20250521135224`. The ink test uses the published `scrollprize/ink_9um` checkpoint. The masking change preserves the scores on measured CT.

## Stream and convert CT

| Problem | Result on PHerc0800 | Evidence |
| --- | --- | --- |
| The renderer fetched CT for the unused part of an edge tile. | A 65 × 67 remote render with the default tile size went from 29 CT chunk fetches to 2 with the same pixel hash. The Windows test used the related reader and cache fixes. | [vesuvius-gui #13](https://github.com/ScrollPrize/vesuvius-gui/pull/13) · [Runs and hashes](https://github.com/mydd7/vesuvius-gui/blob/2531d7c8a47fc75f3d5af1e51eb1722a9eac929e/validation/edge-tiles-validation.json) |
| Gaussian downsampling reflected pixels at each internal chunk boundary. | Changing the chunk size from 8 to 32 changed 2,566 of 11,264 output pixels before the fix and zero afterward. The tests check the converters against a whole-volume reference. | [scroll2zarr #5](https://github.com/ScrollPrize/scroll2zarr/pull/5) · [Before](https://github.com/mydd7/scroll2zarr/blob/4d789c1f1df59be8d91ddcede1c72f1b2afee7c6/validation/gaussian-before.json) · [After](https://github.com/mydd7/scroll2zarr/blob/4d789c1f1df59be8d91ddcede1c72f1b2afee7c6/validation/gaussian-after.json) |
| A shared cache could treat two local volumes named `scan.zarr` in different directories as the same volume. | The second PHerc0800 sample read a voxel value of 137 from the first sample instead of 145 from its own source. After the fix, its render with a reused cache matched its fresh-cache render. Tested with the related Windows build and cache fixes. | [vesuvius-gui #28](https://github.com/ScrollPrize/vesuvius-gui/pull/28) · [Source and render hashes](https://github.com/mydd7/vesuvius-gui/blob/2531d7c8a47fc75f3d5af1e51eb1722a9eac929e/validation/relative-cache-validation.json) |
| PPM export lost valid samples in the last CT slice when interpolation looked for a neighbor beyond the array. | A 24 × 24 export using a PHerc0800 CT crop and planar test geometry recovered all 576 nonzero pixels. The output matched the SciPy reference. | [scroll2zarr #8](https://github.com/ScrollPrize/scroll2zarr/pull/8) · [CLI and interpolation tests](https://github.com/mydd7/scroll2zarr/blob/74fe64ab7ccbd640ea971ab2f5b45bd8b194e19c/validation/ppm-volume-bounds/validation.json) |
| Exporting normalized float CT wrote zeros even though the Zarr metadata still said float32. | The corrected exports preserved fractional and negative values on small PHerc0800 CT derivatives. Four integer exports remained byte-identical. | [scroll2zarr #10](https://github.com/ScrollPrize/scroll2zarr/pull/10) · [Measurements and reproduction](https://github.com/mydd7/scroll2zarr/blob/52d3e6b923478e6c73f181c2a53bddd8d61ffbcb/validation/float-volumes/README.txt) |

## Prepare ink labels with less memory

The PNG-to-Zarr label converter used to allocate 65 planes in memory for one populated input plane. Writing that plane directly and leaving the empty planes to Zarr reduced median peak working set from **545 MiB to 251 MiB**, and median conversion time from **1.665 s to 0.946 s**. All 33 output files matched by SHA-256 across six runs. This benchmark uses one 2,049 × 2,305 crop of the public PHerc. Paris 4 supervision mask on Windows. The PR is a draft. [Villa #2584](https://github.com/ScrollPrize/villa/pull/2584) · [Measurements and hashes](https://github.com/mydd7/villa/blob/a0cffca8041ef6f1085e31ea514515ddeb99d392/validation/png-label-memory/validation.json) · [Reproduction](https://github.com/mydd7/villa/blob/a0cffca8041ef6f1085e31ea514515ddeb99d392/validation/png-label-memory/README.txt)

All eight PRs were open on November 22, 2026. Each link includes the proposed patch and its test details.
