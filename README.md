# SNPE_Yolov5_Model_Conversion

## Prerequisites
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

## Environment Setup
```
conda create -n yolov5 python=3.10
conda activate yolov5
cd yolov5
pip install -r requirements.txt
```

## Convert to ONNX Model
```
cd yolov5
python export.py --weights yolov5s.pt --include torchscript onnx
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

### Model Quantization
```
python ${SNPE_ROOT}/examples/Models/InceptionV3/scripts/create_inceptionv3_raws.py -s 640 -i model_quantization/test_img/ -d model_quantization/output_pictures_640
python ${SNPE_ROOT}/examples/Models/InceptionV3/scripts/create_file_list.py -i model_quantization/output_pictures_640 -o model_quantization/image_file_list.txt -e '*.raw'
snpe-dlc-quantize --input_dlc dlc/yolov5s.dlc --input_list model_quantization/image_file_list.txt --output_dlc dlc/yolov5s_quantized.dlc
```

## Model Visualization
```
snpe-dlc-info -i dlc/yolov5s_quantized.dlc
```

## Model Analysis
```
| 238 | /model.24/Concat_3               | Concat            | /model.24/Reshape_1_output_0 (data type: Float_32; tensor dimension: [1,19200,85]; tensor type: NATIVE)              | output0 (data type: Float_32; tensor dimension: [1,25200,85]; tensor type: APP_READ)                                 | 1x25200x85   | A D G C  | axis: 1                                  |
|     |                                  |                   | /model.24/Reshape_3_output_0 (data type: Float_32; tensor dimension: [1,4800,85]; tensor type: NATIVE)               |                                                                                                                      |              |          | packageName: qti.aisw                    |
|     |                                  |                   | /model.24/Reshape_5_output_0 (data type: Float_32; tensor dimension: [1,1200,85]; tensor type: NATIVE)               |                                                                                                                      |              |          |                                          |
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Note: The supported runtimes column assumes a processor target of Snapdragon 855
Key : A:AIP
      D:DSP
      G:GPU
      C:CPU

------------------------------------------------------------------------------
| Input Name  | Dimensions   | Type      | Encoding Info                     |
------------------------------------------------------------------------------
| images      | 1,640,640,3  | Float_32  | No encoding info for this tensor  |
------------------------------------------------------------------------------
------------------------------------------------------------------------------
| Output Name  | Dimensions  | Type      | Encoding Info                     |
------------------------------------------------------------------------------
| output0      | 1,25200,85  | Float_32  | No encoding info for this tensor  |
------------------------------------------------------------------------------
```
Output Layers:
```
    /model.24/Concat_3
``` 
Output Tensors:
```
    output0
```
