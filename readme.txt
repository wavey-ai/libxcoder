
Quadra Xcoder Codec Library User Guide:



==============================
To configure and build library
==============================

---------------------------
To default install location:
---------------------------

cd libxcoder
./configure && make
sudo make install


Codec library 'libxcoder.a' is by default installed to         /usr/local/lib
Codec library 'libxcoder.so' is by default installed to        /usr/local/lib
Codec Pkg Config 'xcoder.pc' is by default installed to        /usr/local/lib/pkgconfig
Codec API Header 'xcoder_api.h' is by default installed to     /usr/local/include
Standalone test program 'xcoder' is locally generated in       libxcoder/build


--------------------------
To fully customize install:
--------------------------

./configure --libdir=/custom_lib_folder --bindir=/custom_bin_folder \
--includedir=/custom_include_folder --shareddir=/additional_lib_folder && sudo make install


------------
To uninstall:
------------

sudo make uninstall

or

sudo make uninstall LIBDIR=/custom_lib_folder BINDIR=/custom_bin_folder \
INCLUDEDIR=/custom_include_folder SHAREDDIR=/additional_lib_folder


-------------
Build Options:
-------------

To enable video data dump (default: --without-dump)
./configure --with-dump

To enable libxcoder self termination on repeated NVMe error
./configure --with-self-kill

To enable libxcoder latency test patch (default: --without-latency-patch)
./configure --with-latency-patch

To set custom installation path for libxcoder.so and pkgconfig files
./configure --libdir custom_lib_folder/

To set custom installation path for binary utilities (ni_rsrc_mon, etc.)
./configure --bindir custom_bin_folder/

To set custom installation path for libxcoder headers
./configure --includedir custom_include_folder/

To set additional installation path for libxcoder.so
./configure --shareddir additional_lib_folder/

----------------
Peer-to-peer DMA:
----------------
Not supported on Windows or MacOS

To use peer-to-peer DMA, build the NetInt Linux kernel driver (for Linux kernel >= 5.10).
you must have the kernel headers installed prior to compiling with make.

==============================
To run standalone test program
==============================
Note: for now, decoding/encoding/transcoding has the following unsupported features:
 - Audio
 - Container, demuxer, muxer 
 - H.265 frame parsing 
 - Custom user SEI 
 - Encoder sequence change 
 - Decoder/encoder engine reset/recovery 
 - Dolby Vision 
 - HW download (TBC on later date)
 - 2D engine HW frame filtering (TBC on later date)
 - Decoder PPU output configuration (TBC on later date)
 - Decoder multi-output receiving and splitting (TBC on later date)

------------
Test Decoder:
------------

 cd libxcoder/build
 ./xcoder -c 0 -i ../test/akiyo_352x288p25.264 -o aki.yuv -m a2y

------------
Test Encoder:
------------

 cd libxcoder/build
 ./xcoder -c 0 -i ../test/akiyo_352x288p25.yuv -o aki-xcoder.265 -s 352x288 -m y2h
 
---------------
Test Transcoder:
---------------

 cd libxcoder/build
 ./xcoder -c 0 -i ../test/akiyo_352x288p25.264 -o aki-xcoder-trans.265 -m a2h

---------------
Test Transcoder (HW frames):
---------------

 cd libxcoder/build
 ./xcoder -c 0 -i ../test/akiyo_352x288p25.264 -o aki-xcoder-trans.265 -m a2h -d out=hw

---------------
Test Uploader + Encoder:
---------------

 cd libxcoder/build
 ./xcoder -c 0 -i ../test/akiyo_352x288p25.yuv -o aki-xcoder.265 -s 352x288 -m u2h
 
---------------
Test P2P DMA (push method)
---------------
Not supported on Windows or MacOS

This simulates a GPU card writing frames to a Quadra device before encoding.

 cd dma-buf
 make netint_build=eng    # note that the netint_build=eng parameter is important
 sudo insmod netint.ko
 sudo chmod 777 /dev/netint
 cd libxcoder/build
 ./xcoderp2p -c 0 -i ../test/akiyo_352x288p25.yuv -o aki-xcoder.265 -s 352x288 -m p2h

---------------
Test P2P DMA (pull method)
---------------
Not supported on Windows or MacOS

This is a variation of P2P whereby Quadra reads memory from another PCIe device (usually
a GPU card) instead of the PCIe device writing to Quadra memory.

This test program requires TWO Quadra devices. A single T2A card, two T1A cards, or two
T1U cards will suffice.

This test program also requires to be run as root. You will need to run init_rsrc as root.

 cd dma-buf
 make                    # note that we do NOT need the netint_build=eng parameter
                         # for customer A1, there is special build parameter:
                         #    make a1_build=eng
 sudo insmod netint.ko
 cd libxcoder/build
 sudo init_rsrc
 sudo ./xcoderp2p_read -c 1 -g 0 -i bball_720_501.abgr -o bball.h264 -s 1280x720 -m r2a

The "-g 0" will make Quadra device 0 a proxy for a GPU card. The "-c 1" will make Quadra
device 1 read video frames directly from Quadra device 0 before encoding to h264.

To generate raw abgr, you can use ffmpeg to do the conversion from a .264 file:

ffmpeg -c:v h264 -i ../test/1280x720_Basketball.264 -vf format=abgr -c:v rawvideo bball_720_501.abgr

Special build for customer A1
-----------------------------
There is a special customer build parameter for A1.
 make a1_build=eng
This builds the driver with a custom function specific to A1. It will only compile
with their kernel changes.

===================================
To Integrate into user applications
===================================

-------------------
FFmpeg applications:
-------------------

Configure and build FFmpeg with:

./configure --enable-libxcoder && make


------------------
Other applications:
------------------

Codec library: libxcoder.a
API header:     ni_device_api.h

1. Add libxcoder.a as one of libraries to link
2. Add ni_device_api.h in source code calling Codec API

For C
#include "ni_device_api.h"

For C++

extern "C" {
#include "ni_device_api.h"
}
