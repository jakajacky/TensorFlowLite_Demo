# TensorFlowLite_Demo
TensorFlow Lite 的demo。iOS使用TensorFlow Lite配置教程 

# TensorFlow Lite for iOS

## Building

为编译TensorFlow Lite的iOS版静态库, 需要用到MacOS上的终端. 如果还没达标,
那么须先安装 Xcode 8 or later and the tools using `xcode-select`:

```bash
xcode-select --install
```

•第一次安装，需要打开Xcode，按照提示授权信任.

(还需要安装 [Homebrew](http://brew.sh/) installed.)

•另外两个必要工具
[automake](https://en.wikipedia.org/wiki/Automake)/[libtool](https://en.wikipedia.org/wiki/GNU_Libtool):

```bash
brew install automake
brew install libtool
```

•接下来，是运行脚本，下载所需要的依赖，但是先不要立马执行，请看完tips:

```bash
tensorflow/contrib/lite/download_dependencies.sh
```

•这个脚本会联网下载所需依赖包，并放到到这个目录：
`tensorflow/contrib/lite/downloads`.

•demo中需要的label和.tflite模型文件，则下载并解压到：
`tensorflow/contrib/lite/example/ios/camera/data`.

### •tips: 对于国内开发者，可能直接执行脚本，会有几个依赖包下载失败或者 出现问题导致即便下载依赖包步骤通过，但是编译阶段出错，所以我们直接在这里做一些修改：

```bash
EIGEN_URL="$(grep -o 'http.*bitbucket.org/eigen/eigen/get/.*tar\.gz' "${BZL_FILE_PATH}" | grep -v bazel-mirror | head -n1)"
GEMMLOWP_URL="$(grep -o 'https://mirror.bazel.build/github.com/google/gemmlowp/.*zip' "${BZL_FILE_PATH}" | head -n1)"
GOOGLETEST_URL="https://github.com/google/googletest/archive/release-1.8.0.tar.gz"
ABSL_URL="$(grep -o 'https://github.com/abseil/abseil-cpp/.*tar.gz' "${BZL_FILE_PATH}" | head -n1)"
NEON_2_SSE_URL="https://github.com/intel/ARM_NEON_2_x86_SSE/archive/master.zip"
FARMHASH_URL="https://mirror.bazel.build/github.com/google/farmhash/archive/816a4ae622e964763ca0862d9dbd19324a1eaf45.tar.gz"
FLATBUFFERS_URL="https://github.com/google/flatbuffers/archive/master.zip"
# 下面这两个的意思是联网下载。 我们就不要用脚本下载了，这里先注释掉
# MODELS_URL="https://storage.googleapis.com/download.tensorflow.org/models/tflite/mobilenet_v1_1.0_224_ios_lite_float_2017_11_08.zip"
# QUANTIZED_MODELS_URL="https://storage.googleapis.com/download.tensorflow.org/models/tflite/mobilenet_v1_224_android_quant_2017_11_08.zip"
```
```bash
download_and_extract "${EIGEN_URL}" "${DOWNLOADS_DIR}/eigen"
download_and_extract "${GEMMLOWP_URL}" "${DOWNLOADS_DIR}/gemmlowp"
download_and_extract "${GOOGLETEST_URL}" "${DOWNLOADS_DIR}/googletest"
download_and_extract "${ABSL_URL}" "${DOWNLOADS_DIR}/absl"
download_and_extract "${NEON_2_SSE_URL}" "${DOWNLOADS_DIR}/neon_2_sse"
download_and_extract "${FARMHASH_URL}" "${DOWNLOADS_DIR}/farmhash"
download_and_extract "${FLATBUFFERS_URL}" "${DOWNLOADS_DIR}/flatbuffers"
# 下面这两个的意思是下载完依赖包，解压到的路径，同理，也注释掉
# download_and_extract "${MODELS_URL}" "${DOWNLOADS_DIR}/models"
# download_and_extract "${QUANTIZED_MODELS_URL}" "${DOWNLOADS_DIR}/quantized_models"
```
•现在，可以放心的跑脚本了（都在你的TensorFlow的根目录下执行即可）
```bash
tensorflow/contrib/lite/download_dependencies.sh
```
•下载完成，别忘了，上面我们注释掉了两个包的下载，现在需要根据脚本里这两个的下载链接
<a href="https://storage.googleapis.com/download.tensorflow.org/models/tflite/mobilenet_v1_1.0_224_ios_lite_float_2017_11_08.zip">MODELS_URL</a>、
<a href="https://storage.googleapis.com/download.tensorflow.org/models/tflite/mobilenet_v1_224_android_quant_2017_11_08.zip">QUANTIZED_MODELS_URL</a>，
自己去下载并解压。下载完成后，根据注释掉的解压路径，分别把两个文件放到对应的路径下面`tensorflow/contrib/lite/downloads/models`、`tensorflow/contrib/lite/downloads/quantized_models` ，
缺少的文件夹自己创建。

•所需依赖全部放置完毕，接下来就是编译iOS所需要的静态库:

```bash
tensorflow/contrib/lite/build_ios_universal_lib.sh
```
不出意外就是等待编译完成。

最后，编译出的结果放在`tensorflow/contrib/lite/gen/lib`目录下，有各个环境对应的静态库，还有一个lipo合并后的
`tensorflow/contrib/lite/gen/lib/libtensorflow-lite.a`.我们用这个即可。

## 集成到iOS项目中

1、安装 CocoaPods ，这里不做赘述

2、在 tensorflow/contrib/lite/examples/ios/camera路径下 运行 pod install.

3、配置4项（重要）：

  •Target ***->General->Linked Frameworks and Libraries:
  点击＋，添加`tensorflow/contrib/lite/gen/lib`路径下的libtensorflow-lite.a
  
  •Target ***->Build Settings->Library Search Paths：
  
  点击＋，添加`/Users/xiaoqiang/6TensorFlowlite/tensorflow-master/tensorflow/contrib/lite/gen/lib`。
  这个路径我用了绝对路径，开发者根据自己的项目，确保路径链接到此即可。
  
  •Target ***->Build Settings->Header  Search Paths：
  
  点击＋，逐个添加
  
  `'${SRCROOT}/Pods/TensorFlow-experimental/Frameworks/tensorflow_experimental.framework/Headers'`
  
  `'${SRCROOT}/Pods/TensorFlow-experimental/Frameworks/tensorflow_experimental.framework/Headers/third_party/eigen3'`
  
  ` $(inherited)`
  
  `"${PODS_ROOT}/Headers/Public"`
  
  `"${PODS_ROOT}/Headers/Public/TensorFlow-experimental"` 
  
  `"$(SRCROOT)/../../../6TensorFlowlite/tensorflow-master"` 
  
  `"$(SRCROOT)/../../../6TensorFlowlite/tensorflow-master/tensorflow/contrib/lite/downloads"` 
  
  `"$(SRCROOT)/../../../6TensorFlowlite/tensorflow-master/tensorflow/contrib/lite/downloads/flatbuffers/include"`
  
  后三条也使用了我的绝对路径，开发者视自己情况而定，同样确保能最终链接到这些路径。
  
  •Target ***->Build Settings->Other Linker Flags：
  
  点击＋，逐个添加
  
  `$(inherited)` 
  
  `-L` 
  
  `${SRCROOT}/Pods/TensorFlow-experimental/Frameworks/tensorflow_experimental.framework` 
  
  `-ObjC` 
  
  `-l"c++"` 
  
  `-l"protobuf_experimental"` 
  
  `-framework` 
  
  `"Accelerate"` 
  
  `-framework` 
  
  `"tensorflow_experimental"` 
  
  `-force_load` 
  
  `${SRCROOT}/Pods/TensorFlow-experimental/Frameworks/tensorflow_experimental.framework/tensorflow_experimental`

4、打开 tflite_camera_example.xcworkspace, 跑起来吧.
注意：
-   C++11 support (or later) should be enabled by setting `C++ Language Dialect`
    to `GNU++11` (or `GNU++14`), and `C++ Standard Library` to `libc++`.
