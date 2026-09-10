# Autonomous UAV — Detection and TensorRT Benchmarks

YOLO detection scripts for a UAV competition airframe, plus the TensorRT
conversion and FPS measurement work behind deciding what could actually run on
onboard hardware.

> **Status: a working script collection, not an application.** These are the
> individual experiments that fed the integrated systems in
> [AybuHavk](https://github.com/yunusemretom/AybuHavk) and
> [DogFight](https://github.com/yunusemretom/DogFight). Kept for the benchmark
> results.

![demo](docs/demo.gif)

## Tech stack

![Python](https://img.shields.io/badge/Python-3.10-3776AB?logo=python&logoColor=white)
![YOLOv8](https://img.shields.io/badge/YOLOv8-Ultralytics-00FFFF)
![YOLOv5](https://img.shields.io/badge/YOLOv5-Detection-00FFFF)
![TensorRT](https://img.shields.io/badge/TensorRT-FP16_INT8-76B900?logo=nvidia&logoColor=white)
![ONNX](https://img.shields.io/badge/ONNX-Export-005CED?logo=onnx&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-Video_IO-5C3EE8?logo=opencv&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue)

## Quick start

```bash
git clone https://github.com/yunusemretom/Autonomous-UAV.git
cd Autonomous-UAV

python3 -m venv .venv
source .venv/bin/activate
pip install ultralytics opencv-python onnx

python kamera_test.py         # verify the camera opens
python Hiz_deneme_yolo.py     # live detection with FPS readout
python yolov8_deneme.py       # YOLOv8 on a video file
python sabit_qr_yolo.py       # QR detection
python yoloqrson.py           # combined UAV and QR modes, UDP video out
```

TensorRT conversion needs an NVIDIA GPU with TensorRT installed:

```bash
python cevirme_tensorrt.py    # PyTorch -> ONNX -> TensorRT engine
python anadeneme.py           # run the TensorRT engine over a video
```

Training notebooks are `yolov8egitimi.ipynb` and `Veri_Seti_Egitimi.ipynb`. They
need a Roboflow key from the environment:

```bash
export ROBOFLOW_API_KEY=your_key_here
```

## How it works

Each script answers one question, which is why they are separate.

| Script | Question it answers |
|---|---|
| `Hiz_deneme_yolo.py` | what frame rate does this model reach on this hardware |
| `cevirme_tensorrt.py` | how much does TensorRT conversion actually gain |
| `anadeneme.py` | does the converted engine still detect correctly |
| `yoloqrson.py` | can UAV and QR detection share one video pipeline |
| `sabit_qr_yolo.py` | is YOLO better than a QR library for QR codes |

### Why TensorRT instead of a smaller model

The competition constraint was frame rate on an embedded board, not accuracy on
a workstation. The two ways to buy frame rate are a smaller model, which costs
detection quality, or a faster runtime for the same weights, which does not.

`cevirme_tensorrt.py` exports PyTorch to ONNX and then builds a TensorRT engine
with reduced precision. The reason the accuracy check in `anadeneme.py` exists
separately is that quantization is exactly the kind of change that can look free
in a benchmark and quietly cost you small or distant detections, which are the
ones that matter for a target-tracking task. Measuring speed without re-checking
detections would have been measuring the wrong thing.

`Hiz_deneme_yolo.py` reports frame rate and lets the device be chosen, so the
CPU and GPU numbers are directly comparable on the same footage.

### Detecting QR codes with YOLO

`sabit_qr_yolo.py` and the ArUco work in
[Otonom_IHA](https://github.com/yunusemretom/Otonom_IHA) came from the same
finding: dedicated QR libraries expect a reasonably large, reasonably flat,
reasonably well-lit code. From an aircraft the code is small, tilted and motion
blurred, and the library returns nothing rather than something approximate. A
trained detector at least reports a location, which is what a control loop
needs.

## Known limitations

- Scripts, not a package. There is no shared configuration and no entry point.
- Benchmark numbers were taken on one machine on one video and are not
  reproducible from this repository.
- Model weights and result videos are committed, which makes the repository
  large and slow to clone.
- The dataset itself is not included, so training cannot be reproduced.
- File names are Turkish and inconsistent, and several are numbered variants of
  each other.
- No tests.

## Roadmap

Superseded. Integrated versions of this work live in
[AybuHavk](https://github.com/yunusemretom/AybuHavk) and
[DogFight](https://github.com/yunusemretom/DogFight). Remaining cleanup here
would be moving the weights and videos out of git history into release assets.

## License

MIT. See [LICENSE](LICENSE).

Turkish per-script descriptions are preserved in [README.tr.md](README.tr.md).
