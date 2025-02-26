# ffmpeg: enable jpegxs codec

## Linux ffmpeg plugin
As compared with OpenVisualCloud's 
[SVT-JPEG-XS/ffmpeg-plugin/readme](https://github.com/OpenVisualCloud/SVT-JPEG-XS/tree/main/ffmpeg-plugin),
this repository explores SVT-JPEG-XS codec integration into FFmpeg while giving 
details about FFmpeg installation with WSL's Ubuntu 24.04 (Noble Numbat) distribution.

To include as many details as may be required, let us presume that we start with 
a fresh installation of Ubuntu 24.04 (Noble Numbat) distribution into WSL.

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
(Do not forget to save these env vars to your profile, otherwise you have to 
make this exports with each new login.)

But for the FFmpeg installation, we switch to the FFmpeg's guide 
[Compile FFmpeg for Ubuntu, Debian, or Mint](https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu).

#### 3. FFmpeg compile
These are packages required for compiling (we start with a fresh installation of 
Ubuntu!):
```
sudo apt update
sudo apt upgrade
sudo apt install \
  autoconf \
  automake \
  build-essential \
  cmake \
  git-core \
  libass-dev \
  libfreetype6-dev \
  libgnutls28-dev \
  libmp3lame-dev \
  libsdl2-dev \
  libtool \
  libva-dev \
  libvdpau-dev \
  libvorbis-dev \
  libxcb1-dev \
  libxcb-shm0-dev \
  libxcb-xfixes0-dev \
  meson \
  ninja-build \
  pkg-config \
  texinfo \
  wget \
  yasm \
  zlib1g-dev \
  libunistring-dev \
  libaom-dev \
  libdav1d-dev
```
Some components of FFmpeg require, besides yasm, also NASM compiler:
```
sudo apt install nasm
```
Make a new directory to put all of the source code and binaries into:
```
mkdir -p ~/ffmpeg_sources ~/bin
```
All but one external library (Netflix's vmaf) can now be apt installed from
Ubuntu 24.04 distros rather then compiled from sources:
```
sudo apt install libx264-dev
sudo apt install libx265-dev libnuma-dev
sudo apt install libvpx-dev
sudo apt install libfdk-aac-dev
sudo apt install libopus-dev
```
The library libsvtav1 can also be apt installed, only you have to install two
packages:
```
sudo apt install libsvtav1-dev libsvtav1enc-dev
```
FFmpeg calls SvtAv1Enc from libsvtav1enc, but the latter requires libsvtav1
to be installed. The trick is borrowed from ref.
[SvtAv1Enc >= 0.9.0 not found using pkg-config](https://github.com/markus-perl/ffmpeg-build-script/issues/186).
If you like, you can immediately (from the console) verify that a symbol SvtAv1Enc 
is accessible:
```
pkg-config --path SvtAv1Enc
```
I could not find Netflix's vmaf in distros and compiled this lib from the source
(we place vmaf source code in the ffmpeg\_source folder):
```
cd ~/ffmpeg_sources && \
wget https://github.com/Netflix/vmaf/archive/v3.0.0.tar.gz && \
tar xvf v3.0.0.tar.gz && \
mkdir -p vmaf-3.0.0/libvmaf/build &&\
cd vmaf-3.0.0/libvmaf/build && \
meson setup -Denable_tests=false -Denable_docs=false --buildtype=release --default-library=static .. --prefix "$HOME/ffmpeg_build" --bindir="$HOME/bin" --libdir="$HOME/ffmpeg_build/lib" && \
ninja && \
ninja install
```
vmaf is a must for JPEG-XS developers as this component assesses perceptual visual
quality of video. So, while this component is not necessary for demonstration of
FFmpeg SVT-JPEG-XS integration, I recommend you to install it for the perspective.

Now we are ready to download FFmpeg's source code and update some of its files for
compilation with `--enable-libsvtjpegxs` option.

#### 4. Download FFmpeg sources and set env variables 
```
cd ~/ffmpeg_sources
wget -O ffmpeg-snapshot.tar.bz2 https://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2
tar xjvf ffmpeg-snapshot.tar.bz2
cd ffmpeg
PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig"
```
FFmpeg sources are in \~/ffmpeg\_sources, we are ready to make necessary edits. 
OpenVisualCloud's guide applies patches to make these edits; while this procedure
is time- and effort saving, I recommend to manually edit the files to become
familiar with the sources. You can also ignore my advices: to help you, I provide
updated files in the folder `ffmpeg` and its subfolders.

#### 5. What files need to be updated/added to the project

You have to edit the files in the libavcodec folder from the FFmpeg repo:
```
libavcodec/
	codec\_desc.c
	codec\_id.h
	allcodecs.c
	Makefile
```
the files in the libavformat folder:
```
libavformat/
	isom.c
	isom\_tags.c
	movenc.c
```
and
```
configure
```
in the repo root (ffmpeg).

In the folders with my repo you see three type files: for example, 1) `codec_desc.c` 
(updated copy), 2) `codec_desc.c.in` (codec\_desc.c was copied to the file of the 
same name with suffix .in added), 3) `codec_desc_c.diff` (the output of 
`$git diff codec_desc.c codec_desc.c.in`). This \*.diff files are used to automate
applying of patches; you can use these to manually edit the sources.

You have to add two extra files to the `libavcodec` folder, libsvtjpegxs(dec,enc).c.
These two files can be found in the `ffmpeg-plugin` folder of SVT-JPEG-XS repo; 
I put these into the `libavcodec` folder. File libsvtjpegxs(dec,enc).c of my repo 
is edited (added the line #include "libavutil/mem.h"): you should use this edited file.

#### 6. FFmpeg configure
In [Compile FFmpeg for Ubuntu, Debian, or Mint](https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu).
you can see an example of FFmpeg full-fledged configuration. For demonstration of 
FFmpeg integration with SVT-JPEG-XS you can use much simpler ./configure command:
```
$ ./configure --enable-libsvtjpegxs --prefix=$INSTALL_DIR --enable-shared
```
#### 7. make, make install
```
make -j4
make install
```
and enhanced FFmpeg is ready to use!

#### 5 Demonstration
```
wsluser@Computer:~$ ./ffmpeg -i \<videofilename>.mp4 -c:v jpegxs -bpp 1 \<videofilename>_jxs.mkv
wsluser@Computer:~$ ./ffmpeg -ss 00:00:01 -i <videofilename>_jxs.mkv -frames:v 1 -c:v jpegxs -bpp 1 <videofilename>_oneframe.jxs
```
