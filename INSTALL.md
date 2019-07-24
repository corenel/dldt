# Installation on aarch64 platform

## Environment

- Board: NanoPi M4 (a RK3399 SoC based ARM board)
- Soc: Rockchip RK3399
  - CPU: big.LITTLE，Dual-Core Cortex-A72(up to 2.0GHz) + Quad-Core Cortex-A53(up to 1.5GHz)
  - GPU: Mali-T864 GPU，supports OpenGL ES1.1/2.0/3.0/3.1, OpenVG1.1, OpenCL, DX11, and AFBC
  - VPU: 4K VP9 and 4K 10bits H265/H264 60fps decoding, Dual VOP, etc
- Storage: 64GB SD card
- Operating system: Ubuntu 18.04 LTS

## Steps

1. Build CMake (3.10 or higher, 3.14 recommended) and OpenCV (4.1.0) from source. The build script can be found in my repo ([for CMake](https://github.com/corenel/env-setup/blob/master/packages/install_cmake.sh), [for OpenCV](https://github.com/corenel/env-setup/blob/master/packages/install_opencv.sh))

2. Clone this repo and install the dependencies

```bash
$ git clone https://github.com/corenel/dldt.git
$ cd dldt
$ git submodule update --init –-recursive
$ ./inference-engine/install_dependencies.sh
```

3. Build OpenVINO and install

```bash
$ cd inference-engine
$ mkdir build && cd build
$ export OpenCV_DIR=/usr/lib/aarch64-linux-gnu
$ cmake -DCMAKE_BUILD_TYPE=Release \
    -DENABLE_MKL_DNN=OFF \
    -DENABLE_CLDNN=OFF \
    -DENABLE_GNA=OFF \
    -DENABLE_SSE42=OFF \
    -DTHREADING=SEQ \
    ..
$ make
$ sudo make install
```

4. Setup udev rules (maybe a reboot is necessary after this step)

```bash
$ cd /path/to/dldt
$ sudo cp misc/97-myriad-usbboot.rules /etc/udev/rules.d/97-myriad-usbboot.rules
$ sudo service udev restart
$ sudo udevadm control --reload-rules
$ sudo udevadm trigger
$ sudo ldconfig
```

5. Validate the installation (insert your NCS2 before running the following commands)

```bash
$ cd $HOME/Downloads
$ mkdir models && cd models
$ wget https://download.01.org/opencv/2019/open_model_zoo/R1/models_bin/age-gender-recognition-retail-0013/FP16/age-gender-recognition-retail-0013.xml
$ wget https://download.01.org/opencv/2019/open_model_zoo/R1/models_bin/age-gender-recognition-retail-0013/FP16/age-gender-recognition-retail-0013.bin
$ cd /path/to/dldt
$ cd inference-engine/bin/intel64/Release
$ ./benchmark_app \
    -m ~/Downloads/models/age-gender-recognition-retail-0013.xml \
    -i ~/Downloads/setup/president_reagan-62x62.png \
    -d MYRIAD
```

## References

- [Arm 64 Single Board Computers and the Intel® Neural Compute Stick 2 (Intel® NCS 2)](https://software.intel.com/en-us/articles/ARM64-sbc-and-NCS2)
