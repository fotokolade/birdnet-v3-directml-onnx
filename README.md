# birdnet-v3-directml-onnx

**Powered by BirdNET.** The BirdNET+ V3.0 developer preview model (preview 3.1, FP32 ONNX) with
changes that let it run on the ONNX Runtime DirectML provider (Windows, any DirectX 12 GPU).
The files are in the releases of this repository, with their SHA-256. This is not an official
release of the BirdNET team.

| Release | File | Bytes | SHA-256 | Windows on DirectML |
|---|---|---|---|---|
| birdnet-v3-preview3.1-dml-2 | `birdnet-v3.0-preview3.1-global-11k-fp32-dml.onnx` | 541,518,284 | `7537acd8a1402327b1d2693789e3e42eb40cda37c6ebad852a840c63d0ea69d8` | any length |
| birdnet-v3-preview3.1-dml-1 | `birdnet-v3.0-preview3.1-global-11k-fp32-dml.onnx` | 541,598,502 | `1e7fd7c5638c1bf1d6094406a3edf30f50b2211a1e1f5dd7c8a0a700a309fb2b` | 3 s only |

## Original work

BirdNET+ V3.0 model developer preview, preview 3.1, by the K. Lisa Yang Center for Conservation
Bioacoustics (Cornell University), Chemnitz University of Technology and Museum für Naturkunde
Berlin. It builds on Kahl et al. (2021), BirdNET: A deep learning solution for avian diversity
monitoring, and Lasseck (2018), Audio-based Bird Species Identification with Deep Convolutional
Neural Networks.

Lasseck, M., Eibl, M., Klinck, H., & Kahl, S. (2026). BirdNET+ V3.0 model developer preview
(Preview 3.1). Zenodo. https://doi.org/10.5281/zenodo.20703646

Source file: `BirdNET+_V3.0-preview3.1_Global_11K_FP32.onnx` from that Zenodo record (541,598,502
bytes, MD5 `632ffdf0b44048da4876099c2fc676ad` as listed on Zenodo).
Code and documentation of the original: https://github.com/birdnet-team/birdnet-V3.0-dev

## License and terms of use

Licensed under the Creative Commons Attribution-ShareAlike 4.0 International License
(CC BY-SA 4.0), like the original model, see `LICENSE` and
https://creativecommons.org/licenses/by-sa/4.0/. The terms of use of the original model apply
unchanged, see `TERMS_OF_USE.txt`. In particular the model and any derivative must not be used
for poaching or wildlife tracking that facilitates illegal capture, harm, trade or exploitation of
species, and must not be used for any military purpose. Tools that use the model must credit
BirdNET (for example "Powered by BirdNET").

## Changes

Both releases: in the 20 `Reshape` nodes that had the attribute `allowzero=1`, the attribute is set
to `0`. The DirectML provider rejects `allowzero=1`, so the original fails there at its first
`Reshape` (`node_view`). `allowzero` only matters when a target shape contains a 0, and no
`Reshape` of the model has a constant target shape with a 0, so the model computes the same
function.

Release dml-2 only: the 1,023 declared shapes of intermediate tensors (`value_info`) are removed.
They were fixed to 3 s windows (for example `[batch_size, 1, 224, 281]` after the `Resize` node
`node_upsample_bilinear2d`), and DirectML relies on them, so windows of any other length failed
there. ONNX Runtime now infers the shapes itself. Nodes (773), weights (447 initializers), inputs
and outputs are unchanged. The file is made from the dml-1 file, a second run gives the same bytes.

## Checks

All checks against the original TorchScript model `BirdNET+_V3.0-preview3.1_Global_11K_FP32.pt` on
the CPU, all 11,560 classes per window, ONNX Runtime 1.24.4, batch 16.

| Release, window | Material | Windows | Audio hours | Largest difference | Changed top species | Changed detections at 0.1 / 0.15 / 0.25 / 0.5 |
|---|---|---|---|---|---|---|
| dml-1, 3 s (AMD Radeon RX 5700) | 1,832 labelled clips and 6 whole recordings | 36,027 | 30.0 | 2.5e-4 | 0 | 0 / 0 / 0 / 1 of 26,558 |
| dml-2, 3 s (AMD Radeon RX 7900 GRE) | 1,832 labelled clips and 8 whole hours | 39,170 | 32.6 | 2.3e-4 | 0 | 0 / 0 / 0 / 1 of 25,993 |
| dml-2, 5 s (AMD Radeon RX 7900 GRE) | 1,832 labelled clips and 8 whole hours | 23,866 | 33.2 | 1.1e-4 | 0 | 0 / 0 / 0 / 0 of 17,873 |

The material covers 4 recorders, February to June, evening, night and dawn chorus. Control: the
ONNX output compared with the reference of the previous batch changes 30,983 (3 s) and 16,868 (5 s)
detections at 0.15.

## Use

Input `input` [batch, samples] float32: mono audio at 32 kHz. The original analysis script uses 3 s
windows (96,000 samples), dml-2 also runs other lengths on DirectML, for example 5 s (160,000
samples). Outputs `predictions` [batch, 11560] (class probabilities in the order of the original
labels file `BirdNET+_V3.0-preview3.1_Global_11K_Labels.csv` from the same Zenodo record) and
`embeddings` [batch, 1280]. Notes from the checks above: windows of pure digital silence give NaN
(also in the original), and a batch of 16 is about 8 times as fast as a batch of 1 on DirectML.
DirectML runs only the first input shape a session sees at full speed, so keep one window length
and one batch size per session (pad the last batch).

## Disclaimer

THE MODEL IS PROVIDED "AS IS" WITHOUT ANY WARRANTY, as stated in the terms of use of the original
model. This repository is not affiliated with or endorsed by the BirdNET team.
