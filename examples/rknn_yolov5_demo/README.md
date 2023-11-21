# Yolo-v5 demo

# 导出rknn模型
## 使用官方onnx模型

1. 使用yolov5官方仓库导出模型，链接：https://github.com/ultralytics/yolov5, 该demo创建时yolov5的最新节点sha码为: c5360f6e7009eb4d05f14d1cc9dae0963e949213

2. 在yolov5工程的根目录下导出已训练好的yolov5模型，如yolov5s.onnx.

   ```sh
   python export.py --weights yolov5s.pt --img 640 --batch 1 --opset 12
   ```

   注：yolov5工程需要使用pytorch 1.8.0 或 1.9.0 版本才能正常导出。

3. 使用onnx-simplifier工具优化yolov5的onnx模型,安装和优化命令如下:

   ```sh
   ## 如果已安装onnx-simplifier,跳过这句
   pip install onnx-simplifier 
   
   python -m onnxsim yolov5s.onnx  yolov5s.onnx
   ```

### 运行python推理

1. 如果重新导出onnx模型,进入rknn-toolkit2目录，将导出的onnx模型复制到examples/onnx/yolov5目录下yolov5s.onnx，再执行命令:

   ```sh
   cd examples/onnx/yolov5
   python test.py
   ```

### 使用rk预训练模型

由于官方的yolov5s模型中包含了Slice层/Swish层/大kernel_size的MaxPooling层，NPU执行效率不高。我们建议开发者使用NPU友好的算子替换官网的结构，下面给出两个参考网络结构：

1. 改进结构1

```txt
a. 将Focus层改成Conv层
b. 将Swish激活函数改成Relu激活函数
```

我们Demo中提供了一个高性能版本的rknn模型：`model/yolov5s-640-640.rknn`。对应的onnx模型是`convert_rknn_demo/yolov5/onnx_models/yolov5s_rm_transpose.onnx`，该模型是预测80类coco数据集的yolov5s改进结构，将Slice层训练和导出onnx模型过程请参考https://github.com/airockchip/yolov5.git

转换rknn模型的步骤如下：

```sh
cd convert_rknn_demo/yolov5/
python onnx2rknn.py
```

2. 改进结构2

```txt
a. 将Focus层改成Conv层
b. 将Swish激活函数改成Relu激活函数
c. 将大kernel_size的MaxPooling改成3x3 MaxPooling Stack结构
```

训练和导出onnx模型过程请参考请参考https://github.com/EASY-EAI/yolov5

## Android Demo

### 编译

根据指定平台修改`build-android_<TARGET_PLATFORM>.sh`中的Android NDK的路径 `ANDROID_NDK_PATH`，<TARGET_PLATFORM>可以是RK356X或RK3588 例如修改成：

```sh
ANDROID_NDK_PATH=~/opt/tool_chain/android-ndk-r17
```

然后执行：

```sh
./build-android_<TARGET_PLATFORM>.sh
```

### 推送执行文件到板子

连接板子的usb口到PC,将整个demo目录到`/data`:

```sh
adb root
adb remount
adb push install/rknn_yolov5_demo /data/
```

### 运行

```sh
adb shell
cd /data/rknn_yolov5_demo/

export LD_LIBRARY_PATH=./lib
./rknn_yolov5_demo model/ model/yolov5s-640-640.rknn model/bus.jpg
```


## Aarch64 Linux Demo

### 编译

根据指定平台修改`build-linux_<TARGET_PLATFORM>.sh`中的交叉编译器所在目录的路径`TOOL_CHAIN`，例如修改成

```sh
export TOOL_CHAIN=~/opt/tool_chain/gcc-9.3.0-x86_64_aarch64-linux-gnu/host
```

然后执行：

```sh
./build-linux_<TARGET_PLATFORM>.sh
```

### 推送执行文件到板子

连接板子的usb口到PC,将整个demo目录到`/userdata`:

```sh
adb push install/rknn_yolov5_demo_Linux /userdata/
```

### 运行

```sh
adb shell
cd /userdata/rknn_yolov5_demo_Linux/

export LD_LIBRARY_PATH=./lib
./rknn_yolov5_demo model/ model/yolov5s-640-640.rknn model/bus.jpg
```

Note: Try searching the location of librga.so and add it to LD_LIBRARY_PATH if the librga.so is not found on the lib folder. \
Using the following commnands to add to LD_LIBRARY_PATH.
```sh
export LD_LIBRARY_PATH=./lib:<LOCATION_LIBRGA.SO>
```
