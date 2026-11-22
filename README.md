# PHerc0800: rendering and CT data fixes

November 2026 Vesuvius Challenge Progress Prizes · [mydd7](https://github.com/mydd7)

These four patches came from rendering and inspecting public PHerc0800 data. They let a Windows batch render finish, remove predictions outside measured CT, cut the data fetched for small previews, and make the Gaussian pyramid independent of how the input is chunked.

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

The PRs contain the code, test fixtures and reproduction details. All four were open on November 22, 2026.
