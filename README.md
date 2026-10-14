# birdnet-v3-directml-onnx

**Powered by BirdNET.** The BirdNET+ V3.0 developer preview model (preview 3.1, FP32 ONNX) with
one change that lets it run on the ONNX Runtime DirectML provider (Windows, any DirectX 12 GPU).
The file is in the releases of this repository, with its SHA-256. This is not an official release
of the BirdNET team.

| Release | File | Bytes | SHA-256 |
|---|---|---|---|
| birdnet-v3-preview3.1-dml-1 | `birdnet-v3.0-preview3.1-global-11k-fp32-dml.onnx` | 541,598,502 | `1e7fd7c5638c1bf1d6094406a3edf30f50b2211a1e1f5dd7c8a0a700a309fb2b` |

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

In the 20 `Reshape` nodes that had the attribute `allowzero=1`, the attribute is set to `0`.
Nothing else is changed (same size, same weights, same graph). The DirectML provider rejects
`allowzero=1`, so the original fails there at its first `Reshape` (`node_view`). `allowzero`
only matters when a target shape contains a 0, and no `Reshape` of the model has a constant
target shape with a 0, so the model computes the same function.

Checked on 1,832 labelled clips and 6 whole recordings (36,027 windows of 3 s, 30.0 audio hours,
4 recorders, February to June, evening, night, dawn chorus and day; ONNX Runtime 1.24.4,
DirectML, AMD Radeon RX 5700) against the original TorchScript model
`BirdNET+_V3.0-preview3.1_Global_11K_FP32.pt` on the CPU, all 11,560 classes per window: largest
difference of a class probability 2.5e-4, 0 changed top species, 0 changed detections at the
thresholds 0.1, 0.15 and 0.25, 1 changed detection of 26,558 at 0.5.

## Use

Input `input` [batch, samples] float32: mono audio at 32 kHz, 3 s windows (96,000 samples) as in
the original analysis script. Outputs `predictions` [batch, 11560] (class probabilities in the
order of the original labels file `BirdNET+_V3.0-preview3.1_Global_11K_Labels.csv` from the same
Zenodo record) and `embeddings` [batch, 1280]. Notes from the checks above: windows of pure digital
silence give NaN (also in the original), a batch of 16 is about 8 times as fast as a batch of 1 on
DirectML, and windows longer than 3 s fail on DirectML at a `Resize` node.

## Disclaimer

THE MODEL IS PROVIDED "AS IS" WITHOUT ANY WARRANTY, as stated in the terms of use of the original
model. This repository is not affiliated with or endorsed by the BirdNET team.
