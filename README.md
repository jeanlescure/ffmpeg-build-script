# THIS IS A FORK

This fork is not maintained and I modify it indiscriminately.

Find original stable project at: https://github.com/markus-perl/ffmpeg-build-script

## Disclaimer And Data Privacy Notice

This script will download different packages with different licenses from various sources, which may track your usage.
These sources are out of control by the developers of this script. Also, this script can create a non-free and unredistributable binary.
By downloading and using this script, you are fully aware of this.

Use this script at your own risk. I maintain this script in my spare time. Please do not file bug reports for systems
other than Debian and macOS, because I don't have the resources or time to maintain different systems.

## Installation

### Quick install and run (macOS, Linux)

Open your command line and run (curl needs to be installed):

```bash

# Without GPL and non-free codes, see https://ffmpeg.org/legal.html 
$ bash <(curl -s "https://raw.githubusercontent.com/jeanlescure/ffmpeg-build-script/master/web-install.sh?v1")

# With GPL and non-free codes, see https://ffmpeg.org/legal.html 
$ bash <(curl -s "https://raw.githubusercontent.com/jeanlescure/ffmpeg-build-script/master/web-install-gpl-and-non-free.sh?v1")
```

This command downloads the build script and automatically starts the build process.

### Common installation (macOS, Linux)

```bash
$ git clone https://github.com/jeanlescure/ffmpeg-build-script.git
$ cd ffmpeg-build-script
$ ./build-ffmpeg --build
```

## Supported Codecs

* `x264`: H.264 Video Codec (MPEG-4 AVC)
* `x265`: H.265 Video Codec (HEVC)
* `libsvtav1`: SVT-AV1 Encoder and Decoder
* `aom`: AV1 Video Codec (Experimental and very slow!)
* `librav1e`: rust based AV1 encoder (only available if [`cargo` is installed](https://doc.rust-lang.org/cargo/getting-started/installation.html)) 
* `libdav1d`: Fastest AV1 decoder developed by the VideoLAN and FFmpeg communities and sponsored by the AOMedia (only available if `meson` and `ninja` are installed)
* `fdk_aac`: Fraunhofer FDK AAC Codec
* `xvidcore`: MPEG-4 video coding standard
* `VP8/VP9/webm`: VP8 / VP9 Video Codec for the WebM video file format
* `mp3`: MPEG-1 or MPEG-2 Audio Layer III
* `ogg`: Free, open container format
* `vorbis`: Lossy audio compression format
* `theora`: Free lossy video compression format
* `opus`: Lossy audio coding format
* `srt`: Secure Reliable Transport
* `webp`: Image format both lossless and lossy

### HardwareAccel

* `nv-codec`: [NVIDIA's GPU accelerated video codecs](https://devblogs.nvidia.com/nvidia-ffmpeg-transcoding-guide/).
  These encoders/decoders will only be available if a CUDA installation was found while building the binary.
  Follow [these](#Cuda-installation) instructions for installation. Supported codecs in nvcodec:
    * Decoders
        * H264 `h264_cuvid`
        * H265 `hevc_cuvid`
        * Motion JPEG `mjpeg_cuvid`
        * MPEG1 video `mpeg1_cuvid`
        * MPEG2 video `mpeg2_cuvid`
        * MPEG4 part 2 video `mepg4_cuvid`
        * VC-1 `vc1_cuvid`
        * VP8 `vp8_cuvid`
        * VP9 `vp9_cuvid`
    * Encoders
        * H264 `nvenc_h264`
        * H265 `nvenc_hevc`
* `vaapi`: [Video Acceleration API](https://trac.ffmpeg.org/wiki/Hardware/VAAPI). These encoders/decoders will only be
  available if a libva driver installation was found while building the binary. Follow [these](#Vaapi-installation)
  instructions for installation. Supported codecs in vaapi:
    * Encoders
        * H264 `h264_vaapi`
        * H265 `hevc_vaapi`
        * Motion JPEG `mjpeg_vaapi`
        * MPEG2 video `mpeg2_vaapi`
        * VP8 `vp8_vaapi`
        * VP9 `vp9_vaapi`
* `AMF`: [AMD's Advanced Media Framework](https://github.com/GPUOpen-LibrariesAndSDKs/AMF). These encoders will only 
  be available if `amdgpu` drivers are detected in use on the system with `lspci -v`. 
    * Encoders
        * H264 `h264_amf` 


### Apple M1 (Apple Silicon) Support

The script also builds FFmpeg on a new MacBook with an Apple Silicon M1 processor.

### LV2 Plugin Support

If Python is available, the script will build a ffmpeg binary with lv2 plugin support.

## Continuous Integration

ffmpeg-build-script is very stable. Every commit runs against Linux and macOS
with https://github.com/jeanlescure/ffmpeg-build-script/actions to make sure everything works as expected.

## Requirements

### macOS

* XCode 10.x or greater

### Linux

* Debian >= Buster, Ubuntu => Focal Fossa, other Distributions might work too
* Rocky Linux 8
* A development environment and curl is required

```bash
# Debian and Ubuntu
$ sudo apt install build-essential curl

# Fedora
$ sudo dnf install @development-tools curl
```

### Build in Docker (Linux)

With Docker, FFmpeg can be built reliably without altering the host system. Also, there is no need to have the CUDA SDK
installed outside of the Docker image.

##### Default

If you're running an operating system other than the one above, a completely static build may work. To build a full
statically linked binary inside Docker, just run the following command:

```bash
$ docker build --tag=ffmpeg:default --output type=local,dest=build -f Dockerfile .
```

##### CUDA
These builds are always built with the --enable-gpl-and-non-free switch, as CUDA is non-free. See https://ffmpeg.org/legal.html
```bash
export DOCKER_BUILDKIT=1

## Set the DIST (`ubuntu` or `rocky`) and VER (ubuntu: `16.04` , `18.04`, `20.04` or rocky: `8`) environment variables to select the preferred Docker base image.
$ export DIST=centos
$ export VER=8

## Start the build
$ docker build --tag=ffmpeg:cuda-$DIST -f cuda-$DIST.dockerfile --build-arg VER=$VER .
```

Build an `export.dockerfile` that copies only what you need from the image you just built as follows. When running,
move the library in the lib to a location where the linker can find it or set the `LD_LIBRARY_PATH`. Since we have
matched the operating system and version, it should work well with dynamic links. If it doesn't work, edit
the `export.dockerfile` and copy the necessary libraries and try again.

```bash
$ docker build --output type=local,dest=build -f export.dockerfile --build-arg DIST=$DIST .
$ ls build
bin lib
$ ls build/bin
ffmpeg ffprobe
$ ls build/lib
libnppc.so.11 libnppicc.so.11 libnppidei.so.11 libnppig.so.11
```

##### Full static version

If you're running an operating system other than the one above, a completely static build may work. To build a full
statically linked binary inside Docker, just run the following command:

```bash
$ sudo -E docker build --tag=ffmpeg:cuda-static --output type=local,dest=build -f full-static.dockerfile .
```

### Run with Docker (macOS, Linux)

You can also run the FFmpeg directly inside a Docker container.

#### Default - Without CUDA (macOS, Linux)

If CUDA is not required, a dockerized FFmpeg build can be executed with the following command:

```bash
$ sudo docker build --tag=ffmpeg .
$ sudo docker run ffmpeg -i https://files.coconut.co.s3.amazonaws.com/test.mp4 -f webm -c:v libvpx -c:a libvorbis - > test.mp4
```

#### With CUDA (Linux)

To use CUDA from inside the container, the installed Docker version must be >= 19.03. Install the driver
and `nvidia-docker2`
from [here](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#installing-docker-ce).
You can then run FFmpeg inside Docker with GPU hardware acceleration enabled, as follows:

```bash
$ sudo docker build --tag=ffmpeg:cuda -f cuda-ubuntu.dockerfile .
$ sudo docker run --gpus all ffmpeg-cuda -hwaccel cuvid -c:v h264_cuvid -i https://files.coconut.co.s3.amazonaws.com/test.mp4 -c:v hevc_nvenc -vf scale_npp=-1:1080 - > test.mp4
```

### Common build (macOS, Linux)

If you want to enable CUDA, please refer to [these](#Cuda-installation) and install the SDK.

If you want to enable Vaapi, please refer to [these](#Vaapi-installation) and install the driver.

```bash
$ ./build-ffmpeg --build
```

## Cuda installation

CUDA is a parallel computing platform developed by NVIDIA. To be able to compile ffmpeg with CUDA support, you first
need a compatible NVIDIA GPU.

- Ubuntu: To install the CUDA toolkit on Ubuntu, run "sudo apt install nvidia-cuda-toolkit"
- Other Linux distributions: Once you have the GPU and display driver installed, you can follow the
  [official instructions](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)
  or [this blog](https://www.pugetsystems.com/labs/hpc/How-To-Install-CUDA-10-1-on-Ubuntu-19-04-1405/)
  to setup the CUDA toolkit.

## Vaapi installation

You will need the libva driver, so please install it below.

```bash
# Debian and Ubuntu
$ sudo apt install libva-dev vainfo

# Fedora and CentOS
$ sudo dnf install libva-devel libva-intel-driver libva-utils
```

## AMF installation

To use the AMF encoder, you will need to be using the AMD GPU Pro drivers with OpenCL support.
Download the drivers from https://www.amd.com/en/support and install the appropriate opencl versions.

```bash
./amdgpu-pro-install -y --opencl=rocr,legacy
```

## Usage

```bash
Usage: build-ffmpeg [OPTIONS]
Options:
  -h, --help                     Display usage information
      --version                  Display version information
  -b, --build                    Starts the build process
      --enable-gpl-and-non-free  Enable non-free codecs  - https://ffmpeg.org/legal.html
      --latest                   Build latest version of dependencies if newer available
  -c, --cleanup                  Remove all working dirs
      --full-static              Complete static build of ffmpeg (eg. glibc, pthreads etc...) **only Linux**
                                 Note: Because of the NSS (Name Service Switch), glibc does not recommend static links.
```

## Notes of static link

- Because of the NSS (Name Service Switch), glibc does **not recommend** static links. See detail
  below: https://sourceware.org/glibc/wiki/FAQ#Even_statically_linked_programs_need_some_shared_libraries_which_is_not_acceptable_for_me.__What_can_I_do.3F

- The libnpp in the CUDA SDK cannot be statically linked.
- Vaapi cannot be statically linked.
