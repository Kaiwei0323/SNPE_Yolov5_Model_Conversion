# SNPE_Yolov5_Model_Conversion

## Prerequisites
* Hardware Platform: QCS6490
* OS: Ubuntu 22.04
* SNPE SDK: v2.22.6.240515
* Model: YOLOv5s.pt

## SNPE SDK Installation
[SNPE SDK](https://www.qualcomm.com/developer/software/neural-processing-sdk-for-ai)

[ANDROID NDK](https://dl.google.com/android/repository/android-ndk-r26c-linux.zip)

## Install YOLOv5 Repo
```
git clone https://github.com/ultralytics/yolov5.git
```

## Replace Files in YOLOv5 Repo
```
yolov5
L---> export.py
L---> models
  L---> yolo.py
```

## Navigate to YOLOv5 Folder and Convert the Model
```
python3 export.py --weights yolov5.pt --optimize --opset 11 --simplify --include onnx
```

## Setup SNPE Environment and Convert ONNX Model to DLC
### SNPE Environment
```
conda create --name snpe python=3.10
conda activate snpe
export SNPE_ROOT=/home/inventec/Documents/Qualcomm/model_conversion/v2.22.6.240515/qairt/2.22.6.240515
${SNPE_ROOT}/bin/check-python-dependency
sudo bash ${SNPE_ROOT}/bin/check-linux-dependency.sh
sudo apt-get install make
```
### Android NDK
```
export ANDROID_NDK_ROOT=/home/inventec/Documents/Qualcomm/model_conversion/android-ndk-r26c-linux/android-ndk-r26c
export PATH=${ANDROID_NDK_ROOT}:${PATH}
```

### Environment Check
```
${SNPE_ROOT}/bin/envcheck -c
```

### ONNX Installation
```
pip install onnx==1.12.0
pip install onnxruntime==1.17.1
```
Reference:
[ONNX](https://pypi.org/project/onnx/1.12.0/)
[ONNX RUNTIME](https://pypi.org/project/onnxruntime/1.17.1/)

### Environment Setup
```
source ${SNPE_ROOT}/bin/envsetup.sh
```

### Model Conversion
```
snpe-onnx-to-dlc --input_network models/yolov5s.onnx --output_path dlc/yolov5s.dlc
```
