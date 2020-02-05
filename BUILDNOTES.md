# Windows for C#

 * CUDA/cuDNN
   * CUDA https://developer.nvidia.com/cuda-downloads e.g. http://developer.download.nvidia.com/compute/cuda/10.2/Prod/network_installers/cuda_10.2.89_win10_network.exe
   * `--cuda_home "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.2"`
   * cuDNN https://developer.nvidia.com/cudnn this requires membership unfortunately but e.g. https://developer.nvidia.com/compute/machine-learning/cudnn/secure/7.6.5.32/Production/10.2_20191118/cudnn-10.2-windows10-x64-v7.6.5.32.zip for Windows 10 or https://developer.nvidia.com/compute/machine-learning/cudnn/secure/7.6.5.32/Production/10.2_20191118/cudnn-10.2-windows7-x64-v7.6.5.32.zip or Linux (general release) https://developer.nvidia.com/compute/machine-learning/cudnn/secure/7.6.5.32/Production/10.2_20191118/cudnn-10.2-linux-x64-v7.6.5.32.tgz
   * Extract to some folder (here named `cuda` change to `cudnn`)
   * `--cudnn_home C:\git\nvidia\cuda` 
 * TensorRT
   * https://developer.nvidia.com/tensorrt this requires membership too.
     * 6.0.1.5: Windows 10 https://developer.nvidia.com/compute/machine-learning/tensorrt/secure/6.0/GA_6.0.1.5/zips/TensorRT-6.0.1.5.Windows10.x86_64.cuda-10.1.cudnn7.6.zip or Linux https://developer.nvidia.com/compute/machine-learning/tensorrt/secure/6.0/GA_6.0.1.8/tars/TensorRT-6.0.1.8.Ubuntu-18.04.x86_64-gnu.cuda-10.2.cudnn7.6.tar.gz 
     * 7.0.0.11: Windows 10 https://developer.nvidia.com/compute/machine-learning/tensorrt/secure/7.0/7.0.0.11/zips/TensorRT-7.0.0.11.Windows10.x86_64.cuda-10.2.cudnn7.6.zip or Linux https://developer.nvidia.com/compute/machine-learning/tensorrt/secure/7.0/7.0.0.11/tars/TensorRT-7.0.0.11.Ubuntu-18.04.x86_64-gnu.cuda-10.2.cudnn7.6.tar.gz 
     * 
   * `--use_tensorrt`
   * `--tensorrt_home C:\git\nvidia\TensorRT-7.0.0.11`
 * DNNL
   * `--use_dnnl` (seems like onnx runtime then downloads source) otherwise see below.
   * Download latest binaries from https://github.com/intel/mkl-dnn/releases
   * We choose vcomp here for **Microsoft Visual C OpenMP runtime** e.g.
     `dnnl_win_1.2.0_cpu_vcomp.zip`
   * Extract to e.g. `C:\git\intel\dnnl_win_1.2.0_cpu_vcomp`

 * Add `--skip_submodule_sync` if sub-modules already synced.
 * Add `--skip_tests` if skipping tests that can take a long time.

In ONNX runtime directory e.g. `C:\git\oss\onnxruntime` run the following from **Developer Command Prompt for VS 2017**:
For parameters see https://github.com/microsoft/onnxruntime/blob/master/tools/ci_build/build.py
```
./build.bat  --cmake_path "C:\Program Files\CMake\bin\cmake.exe" --config RelWithDebInfo --build_shared_lib --build_csharp --parallel --use_cuda --cuda_version 10.2 --cuda_home "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.2" --cudnn_home C:\git\nvidia\cuda --use_tensorrt --tensorrt_home C:\git\nvidia\TensorRT-7.0.0.11 --use_dnnl
```

VS2019
```
C:\git\oss\ort>build.bat  --cmake_path "C:\Program Files\CMake\bin\cmake.exe" --config RelWithDebInfo --build_shared_lib --build_csharp --parallel --use_cuda --cuda_version 10.2 --cuda_home "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.2" --cudnn_home C:\git\nvidia\cuda --use_tensorrt --tensorrt_home C:\git\nvidia\TensorRT-7.0.0.11 --use_dnnl --cmake_generator "Visual Studio 16 2019" --skip_tests
```

`--cuda_version 10.1´ due to cmake issues see below.
```
build.bat  --cmake_path "C:\Program Files\CMake\bin\cmake.exe" --config RelWithDebInfo --build_shared_lib --build_csharp --parallel --use_cuda --cuda_version 10.1 --cuda_home "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.2" --cudnn_home C:\git\nvidia\cuda --use_tensorrt --tensorrt_home C:\git\nvidia\TensorRT-7.0.0.11 --use_dnnl
```

## CUDA 10.2 Issues
https://github.com/microsoft/onnxruntime/issues/1859#issuecomment-581390245
switch to 10.1 but supply the 10.2 path, then fix delay loading...

These issue seem to relate to VS2017 shipping with an old CMake version (in **Developer Command Prompt for VS 2017**):
```
C:\Program Files (x86)\Microsoft Visual Studio\2017\Professional>cmake --version
cmake version 3.12.18081601-MSVC_2
CMake suite maintained and supported by Kitware (kitware.com/cmake).
```
Ways to override can be seen at:
https://stackoverflow.com/questions/49221297/use-installed-cmake-instead-of-embedded-one-in-visual-studio-2017
Which suggest replacing build in:
```
ren "C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake" _CMake
mklink /d "C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake" "C:\Program Files\CMake"
```

## Delay Load Issues
Delay loading does not always appear to have been defined correctly (possible due to build script issues/lack of specifying appropriate version etc.):
```
LINK : warning LNK4199: /DELAYLOAD:cudart64_102.dll ignored; no imports found from cudart64_102.dll [C:\git\oss\onnxruntime\build\Windows\RelWithDebInfo\onn
xruntime.vcxproj]
LINK : error LNK1218: warning treated as error; no output file generated [C:\git\oss\onnxruntime\build\Windows\RelWithDebInfo\onnxruntime.vcxproj]`
```
Overall we want to delay load any dlls from a specific execution provider, to allow onnxruntime.dll to run without them.