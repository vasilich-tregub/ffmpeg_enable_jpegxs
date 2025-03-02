# FFMPEG-based VfW codec 

FFmpeg can be used in a video compressor manager for VfW solutions. In fact, the problem 
of integrating newly developed media formats with VfW-based projects was solved by the 
very popular solution [x264vfw](https://github.com/MasterNobody/x264vfw) as early as the 
year of 2010.

Certainly, the integration of FFmpeg into VfW solutions per se won't help push the 
legacy solution performance and features above limits imposed by the VfW framework, 
but, inviting CI/CD pipeline into the solution, it improves the maintentability.

The x264vfw project is 15 years old and is so well written that it can be used 
'plug-and-play' in the year of 2025. The development tools and environment, however, 
are changing at a much greater pace, and the guide might become useful for developers, 
especially for those who come into industry recently.

As this is a makefile project, beginners often seek guidance on how to use this project 
in Visual Studio. The straightforward answer is "use Developer Command console for 
VS2022 and nmake the project", and it would be the whole truth if the only purpose would
be to build the project. In view of possible interaction with other media-related 
projects, I recommend use the MSYS2, Software Distribution and Building Platform 
for Windows.

Use MSYS2 UCRT console to build the universal c runtime solution.

To build an FFMPEG-based VfW codec, you need to build three projects: 
* [FFMPEG](https://trac.ffmpeg.org/wiki/CompilationGuide/MinGW)
* [x264](https://github.com/mirror/x264)
* and [x264vfw](https://github.com/MasterNobody/x264vfw) itself.

You have also to set environment variables to tell x264vfw makefile where to find source, 
build, and library directories of attached projects:
```
export FFMPEG_DIR=/home/User/ffmpeg
export FFMPEG_NAME=ffmpeg
export FFMPEG_BUILD_DIR=$FFMPEG_DIR/ffbuild
export X264_DIR=/home/User/x264-master
```
You better save this exports in your .bash-profile file.

The `make` fails because of deprecated functions that ffmeg themself declared deprecated;
the remedy is simple: follow recommendations given in the make printout and comment out 
the offending functions.

The other minor problem is a missing libswresample library. For some unknown reason, 
the line
```
VFW_LDFLAGS += "-L$(FFMPEG_DIR)/libswresample" -lswresample $(EXTRALIBS-swresample)
```
in the x264vfw's Makefile is commented. Uncomment this line, and the build succeeds.

You have got a fully customizable VfW codec although you do not write a line of code.

Now, you can add a new codec (jpegxs, for example) into FFMPEG, recompile x264vfw 
(maybe renaming the project is a good idea), and use it in your VfW-based application 
to work with JPEG XS media streams.
