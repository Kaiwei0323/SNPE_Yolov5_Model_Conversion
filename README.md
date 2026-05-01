# SNPE_Model_Conversion

## Prerequisites
* OS: Ubuntu 22.04 (X86)
* SNPE SDK: v2.22.6.240515
* Supported Model: YOLOV8, YOLOV11, DETR-Resnet101

## SNPE SDK Installation
[SNPE SDK](https://www.qualcomm.com/developer/software/neural-processing-sdk-for-ai)

[ANDROID NDK](https://dl.google.com/android/repository/android-ndk-r26c-linux.zip)

## Model Repo
[YOLOV8](https://docs.ultralytics.com/models/yolov8/)

[YOLOV11](https://docs.ultralytics.com/models/yolo11/)

[DETR-Resnet101](https://aihub.qualcomm.com/models/detr_resnet101)

## File Tree
```
Documents
L---> v2.22.6.240515
L---> android-ndk-r26c-linux
```

## Setup SNPE Environment and Convert ONNX Model to DLC
1. Set Up SNPE Environment
```
conda create --name snpe python=3.10
conda activate snpe
export SNPE_ROOT=/home/inventec/Documents/v2.22.6.240515/qairt/2.22.6.240515
${SNPE_ROOT}/bin/check-python-dependency
sudo bash ${SNPE_ROOT}/bin/check-linux-dependency.sh
sudo apt-get install make
```
2. Set Up Android NDK
```
export ANDROID_NDK_ROOT=/home/inventec/Documents/android-ndk-r26c-linux/android-ndk-r26c
export PATH=${ANDROID_NDK_ROOT}:${PATH}
```

3. Check Environment
```
${SNPE_ROOT}/bin/envcheck -c
```

4. Install ONNX and ONNX Runtime
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

### YOLOV8 Model Conversion
1. Create the activation encoding file (act.encodings) with following content:
```
{
    "activation_encodings": {
        "/model.22/Sigmoid_output_0": [
            {
                "bitwidth": 16,
                "dtype": "int",
                "max": 0.996093750000,
                "min": 0.0,
                "offset": 0,
                "scale": 0.00001519941634
            }
        ],
        "output0": [
            {
                "bitwidth": 16,
                "dtype": "int",
                "is_symmetric": "False",
                "max": 643.878662109375,
                "min": 0.0,
                "offset": 0,
                "scale": 0.00982495860394255
            }
        ]
    },
    "param_encodings": {
    
    }
}
```
2. Convert ONNX to DLC
```
snpe-onnx-to-dlc --input_network models/yolov8s.onnx --quantization_overrides act.encodings --output_path dlc/yolov8s.dlc
```

3. Model Quantization
* Prepare dataset for quantization
```
python ${SNPE_ROOT}/examples/Models/InceptionV3/scripts/create_inceptionv3_raws.py -s 640 -i model_quantization/test_img/ -d model_quantization/output_pictures_640
python ${SNPE_ROOT}/examples/Models/InceptionV3/scripts/create_file_list.py -i model_quantization/output_pictures_640 -o model_quantization/image_file_list.txt -e '*.raw'
snpe-dlc-quantize --input_dlc dlc/yolov8s.dlc --override_params --input_list model_quantization/image_file_list.txt --output_dlc dlc/yolov8s_quantized.dlc
```

### DETR-Resnet101 Model Conversion
1. Convert ONNX to DLC
```
snpe-onnx-to-dlc --input_network models/detr-resnet101.onnx --output_path dlc/detr-resnet101.dlc
```
2. Model Quantization
* Prepare dataset for quantization
```
python ${SNPE_ROOT}/examples/Models/InceptionV3/scripts/create_inceptionv3_raws.py -s 480 -i model_quantization/test_img/ -d model_quantization/output_pictures_480
python ${SNPE_ROOT}/examples/Models/InceptionV3/scripts/create_file_list.py -i model_quantization/output_pictures_480 -o model_quantization/image_file_list.txt -e '*.raw'
snpe-dlc-quantize --input_dlc detr-resnet101.dlc --input_list model_quantization/image_file_list.txt --output_dlc dlc/detr-resnet101_quantized.dlc
```

## Model Visualization
```
snpe-dlc-info -i dlc/detr-resnet101_quantized.dlc
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

### Reference
* https://docs.qualcomm.com/bundle/publicresource/topics/80-63442-2/model_conv_onnx.html
* https://docs.qualcomm.com/doc/80-63442-10/topic/setup_linux.html
