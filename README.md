# tflite_runtime_on_windows
在Windows上编译tflite_runtime

### 安装Microsoft Visual Studio

https://visualstudio.microsoft.com/zh-hans/vs/



### MSYS2安装

安装包下载链接：https://github.com/msys2/msys2-installer/releases/download/2024-12-08/msys2-x86_64-20241208.exe

官方教程：https://www.msys2.org/#installation

添加系统环境变量：

```
MSYS2_PATH_TYPE=inherit
```

在环境变量 Path 中也将 MSYS2 的二进制目录 `D:\Softwares\msys64\usr\bin` 添加进去，这里 `D:\Softwares\msys64` 替换成你自己的安装路径。



### bazel 安装

官方教程参考：https://bazel.google.cn/versions/6.5.0/install/windows?hl=zh-cn

版本：`6.5.0`（高版本移除了WORKSPACE，会引发报错）

下载链接：https://github.com/bazelbuild/bazel/releases/tag/6.5.0

文件名：`bazel-6.5.0-windows-x86_64.exe`

此文件为编译好的可执行文件，无需安装。将其改名为 bazel.exe，直接将其所在的目录添加到系统环境变量 Path 中。然后可以在 cmd 中执行以下命令检查是否安装成功：

```bash
bazel version
```

为 bash 和 Visual C++ 设置系统环境变量：

```
BAZEL_SH=D:\Softwares\msys64\usr\bin\bash.exe
BAZEL_VC=C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC
```



### python环境准备

```sh
conda create -n tflite_runtime python=3.10
pip install numpy==1.26.4
pip install six wheel pybind11
pip install keras_applications==1.0.6 --no-deps
pip install keras_preprocessing==1.0.5 --no-deps
sh tensorflow/lite/tools/pip_package/build_pip_package_with_cmake.sh
```



### 编译 tflite_runtime

1. 下载 tensorflow 源码：https://github.com/tensorflow/tensorflow/archive/refs/tags/v2.18.0.zip

2. 解压源码，然后修改源码 `tensorflow-2.18.0\tensorflow\lite\kernels\stablehlo_reduce_window.cc`，在372行处插入空行。

   ```
   ...
   // │X│X X│X│
   // └─┘   └─┘
   
   template <class Op, class Type>
   void ReduceWindowImpl(const Type* input, Type* output,
   ...
   ```

3. 在设置（系统 > 开发者选项）中开启发人员模式

4. 在 Windows 开始菜单中找到 MSYS2 UCRT64，鼠标右键然后以管理员身份运行，此时会进入到 MYSY2 的终端窗口。

5. 在 MYSY2 终端窗口中执行以下命令设置 python 版本：

   ```bash
   export HERMETIC_PYTHON_VERSION=3.10
   ```

6. 激活 python 环境

   ```bash
   source  activate tflite_runtime
   ```

7. 进入到 tensorflow 源码目录，下面命令中的地址需要更改为你的自己的源码路径

   ```bash
   cd /d/Workspace/tensorflow-2.18.0/
   ```

8. 执行编译命令

   ```bash
   sh tensorflow/lite/tools/pip_package/build_pip_package_with_bazel.sh windows
   ```

9. 获取结果：`"tensorflow\lite\tools\pip_package\gen\tflite_pip\python3\dist\` 目录下的 `tflite_runtime-2.18.0-cp310-cp310-win_amd64.whl` 即为编译结果。

   下载：[tflite_runtime-2.18.0-cp310-cp310-win_amd64.whl](./tflite_runtime-2.18.0-cp310-cp310-win_amd64.whl)

   

### 参考

https://www.tensorflow.org/lite/guide/build_cmake_pip?hl=zh-cn

https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/tools/pip_package

https://github.com/NexelOfficial/tflite-runtime-win

https://github.com/tensorflow/tensorflow/issues/64376#issuecomment-2069774768
