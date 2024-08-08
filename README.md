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
| 197 | /model.24/Sigmoid               | ElementWiseNeuron | onnx::Sigmoid_329 (data type: Float_32; tensor dimension: [1,80,80,21]; tensor type: APP_READ)                       | output0 (data type: Float_32; tensor dimension: [1,80,80,21]; tensor type: APP_READ)                                 | 1x80x80x21   | A D G C  | operation: 6                             |
|     |                                 |                   |                                                                                                                      |                                                                                                                      |              |          | packageName: qti.aisw                    |
| 198 | /model.24/m.1/Conv              | Conv2d            | /model.20/cv3/act/Mul_output_0 (data type: Float_32; tensor dimension: [1,40,40,256]; tensor type: NATIVE)           | onnx::Sigmoid_331 (data type: Float_32; tensor dimension: [1,40,40,21]; tensor type: APP_READ)                       | 1x40x40x21   | A D G C  | bias_op_name: model.24.m.1.bias          |
|     |                                 |                   | model.24.m.1.weight (data type: Float_32; tensor dimension: [1,1,256,21]; tensor type: STATIC)                       |                                                                                                                      |              |          | dilation: [1, 1]                         |
|     |                                 |                   | model.24.m.1.bias (data type: Float_32; tensor dimension: [21]; tensor type: STATIC)                                 |                                                                                                                      |              |          | group: 1                                 |
|     |                                 |                   |                                                                                                                      |                                                                                                                      |              |          | packageName: qti.aisw                    |
|     |                                 |                   |                                                                                                                      |                                                                                                                      |              |          | pad_amount: [[0, 0], [0, 0]]             |
|     |                                 |                   |                                                                                                                      |                                                                                                                      |              |          | padding_size_strategy: 5                 |
|     |                                 |                   |                                                                                                                      |                                                                                                                      |              |          | stride: [1, 1]                           |
|     |                                 |                   |                                                                                                                      |                                                                                                                      |              |          | param count: 5k (0.0769%)                |
|     |                                 |                   |                                                                                                                      |                                                                                                                      |              |          | MACs per inference: 8M (0.109%)          |
| 199 | /model.24/Sigmoid_1             | ElementWiseNeuron | onnx::Sigmoid_331 (data type: Float_32; tensor dimension: [1,40,40,21]; tensor type: APP_READ)                       | 332 (data type: Float_32; tensor dimension: [1,40,40,21]; tensor type: APP_READ)                                     | 1x40x40x21   | A D G C  | operation: 6                             |
|     |                                 |                   |                                                                                                                      |                                                                                                                      |              |          | packageName: qti.aisw                    |
| 200 | /model.24/m.2/Conv              | Conv2d            | /model.23/cv3/act/Mul_output_0 (data type: Float_32; tensor dimension: [1,20,20,512]; tensor type: NATIVE)           | onnx::Sigmoid_333 (data type: Float_32; tensor dimension: [1,20,20,21]; tensor type: APP_READ)                       | 1x20x20x21   | A D G C  | bias_op_name: model.24.m.2.bias          |
|     |                                 |                   | model.24.m.2.weight (data type: Float_32; tensor dimension: [1,1,512,21]; tensor type: STATIC)                       |                                                                                                                      |              |          | dilation: [1, 1]                         |
|     |                                 |                   | model.24.m.2.bias (data type: Float_32; tensor dimension: [21]; tensor type: STATIC)                                 |                                                                                                                      |              |          | group: 1                                 |
|     |                                 |                   |                                                                                                                      |                                                                                                                      |              |          | packageName: qti.aisw                    |
|     |                                 |                   |                                                                                                                      |                                                                                                                      |              |          | pad_amount: [[0, 0], [0, 0]]             |
|     |                                 |                   |                                                                                                                      |                                                                                                                      |              |          | padding_size_strategy: 5                 |
|     |                                 |                   |                                                                                                                      |                                                                                                                      |              |          | stride: [1, 1]                           |
|     |                                 |                   |                                                                                                                      |                                                                                                                      |              |          | param count: 10k (0.154%)                |
|     |                                 |                   |                                                                                                                      |                                                                                                                      |              |          | MACs per inference: 4M (0.0546%)         |
| 201 | /model.24/Sigmoid_2             | ElementWiseNeuron | onnx::Sigmoid_333 (data type: Float_32; tensor dimension: [1,20,20,21]; tensor type: APP_READ)                       | 334 (data type: Float_32; tensor dimension: [1,20,20,21]; tensor type: APP_READ)                                     | 1x20x20x21   | A D G C  | operation: 6                             |
|     |                                 |                   |                                                                                                                      |                                                                                                                      |              |          | packageName: qti.aisw                    |
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
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
------------------------------------------------------------------------------------
| Output Name        | Dimensions  | Type      | Encoding Info                     |
------------------------------------------------------------------------------------
| onnx::Sigmoid_331  | 1,40,40,21  | Float_32  | No encoding info for this tensor  |
| 332                | 1,40,40,21  | Float_32  | No encoding info for this tensor  |
| 334                | 1,20,20,21  | Float_32  | No encoding info for this tensor  |
| onnx::Sigmoid_333  | 1,20,20,21  | Float_32  | No encoding info for this tensor  |
| onnx::Sigmoid_329  | 1,80,80,21  | Float_32  | No encoding info for this tensor  |
| output0            | 1,80,80,21  | Float_32  | No encoding info for this tensor  |
------------------------------------------------------------------------------------
```
Output Layers:
```
    /model.24/Sigmoid
    /model.24/Sigmoid_1
    /model.24/Sigmoid_2
``` 
Output Tensors:
```
    output0
    332
    334
```
