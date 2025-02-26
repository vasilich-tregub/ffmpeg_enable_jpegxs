# ffmpeg: enable jpegxs codec

## Linux ffmpeg plugin
As compared with OpenVisualCloud's 
[SVT-JPEG-XS/ffmpeg-plugin/readme](https://github.com/OpenVisualCloud/SVT-JPEG-XS/tree/main/ffmpeg-plugin),
this repository explores SVT-JPEG-XS codec integration into FFmpeg while giving 
details about FFmpeg installation with WSL's Ubuntu 24.04 (Noble Numbat) distribution.

For consistency of the narrative, I repeat the first three items of OVC's document:
#### 0. Create installation directory and export env variable:
```
mkdir install-dir
export INSTALL_DIR=$PWD/install-dir
```
#### 1. Compile and install svt-jpegxs:
```
cd <jpeg-xs-repo>/Build/linux
./build.sh install --prefix $INSTALL_DIR
```
#### 2. Export installation location:
```
export LD_LIBRARY_PATH="$INSTALL_DIR/lib:${LD_LIBRARY_PATH}"
export PKG_CONFIG_PATH="$INSTALL_DIR/lib/pkgconfig:${PKG_CONFIG_PATH}"
```
But for the FFmpeg installation, switch to the FFmpeg's guide 
[Compile FFmpeg for Ubuntu, Debian, or Mint](https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu).
