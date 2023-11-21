下述<TARGET_PLATFORM>是RK356X或RK3588
# Aarch64 Linux Demo
## build

modify `GCC_COMPILER` on `build-linux.sh` for target platform, then execute

```
./build-linux.sh
```

## install

connect device and push build output into `/userdata`

```
adb push install/rknn_multiple_input_demo_Linux /userdata/
```

## run

```
adb shell
cd /userdata/rknn_multiple_input_demo_Linux/
```

```
export LD_LIBRARY_PATH=./lib
./rknn_multiple_input_demo model model/multiple_input_demo.rknn model/input1.bin#model/input2.bin
```

# Android Demo
## build

modify `ANDROID_NDK_PATH` on `build-android.sh` for target platform, then execute

```
./build-android.sh
```

## install

connect device and push build output into `/data`

```
adb push install/rknn_multiple_input_demo_Android /data/
```

## run

```
adb shell
cd /data/rknn_multiple_input_demo_Android/
```

```
export LD_LIBRARY_PATH=./lib
./rknn_multiple_input_demo model/ model/multiple_input_demo.rknn model/input1.bin#model/input2.bin
```
