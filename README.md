This repo is for Archlinux AARCH64 like OS. This is based upon https://github.com/raspberrypi/libcamera-apps

# libcamera-apps

This is a small suite of libcamera-based apps that aim to copy the functionality of the existing "raspicam" apps. They are currently still very much a work in progress!

Binary provided:

* libcamera-still - a libcamera version of raspistill.
* libcamera-vid - a libcamera version of raspivid.
* libcamera-raw - a version of libcamera-vid that saves a file of uncompressed raw (Bayer) video.
* libcamera-hello - a new and very small app that aims to show pretty much the easiest way to get frames from the camera onto the display.
* libcamera-jpeg - a cut-down version of libcamera-still that runs a preview and captures a JPEG, without the distraction of all those other options.

License
-------

The source code is made available under the simplified [BSD 2-Clause license](https://spdx.org/licenses/BSD-2-Clause.html).

Building
--------

libcamera-apps are supported for Pi 4 based platforms.

Instructions for installing libcamera-apps and after all its dependencies were installed.

Also make sure you have the correct _dtoverlay_ for your sensor in the /boot/config.txt file (for example, `dtoverlay=imx477` for the HQ cam) and then reboot your Pi.

#### libcamera-git (AUR)

Install this package from [AUR](https://aur.archlinux.org/packages/libcamera-git/)

At this stage you may wish to check that qcam works. Type `build/src/qcam/qcam` and check that you see a camera image.

*Note*

On some lower memory platforms (e.g. 1GB) there have been cases of ninja exhausting all the system memory and aborting (as it will attempt to use all the CPU cores). If this happens, please try replacing `ninja -C build` by `ninja -C build -j 2` - this will restrict Ninja to only 2 cores.

#### libepoxy (extra)

Install this package from extra repo.

libcamera-apps requires libepoxy to be installed.

#### libcamera-apps

libcamera-apps further requires the following:

cmake make libepoxy boost-libs libdrm libexif


To check everything is working correctly, type `./libcamera-hello` - you should see a preview window displayed for about 5 seconds.

*Note for devices with few RAM*

1GB devices may need `make -j2` instead of `make -j4`.

Please ensure you have `dtoverlay=vc4-fkms-v3d` in your `/boot/config.txt` file.

Understanding the Applications
------------------------------

All the applications support the `--help` option.

Notes
-----

Mostly these apps behave like their raspi- counterparts, however not everything is exactly identical. Below we outline the principal differences that existing users of raspistill/vid will notice.

#### General

* The use of Boost program_options doesn't allow multi-character short versions of options, so where these were present I've had to drop them. The long form options are named the same, and any single character short forms are preserved.

* libcamera-still and libcamera-jpeg don't currently show the capture image.

* libcamera performs its own camera mode selection, so the `--mode` option is not supported. It deduces camera modes from the resolutions requested.

* The following features of the raspicam apps are not supported as the code would have to be implemented on the ARM now:
  * opacity (`--opacity`)
  * image effects (`--imxfx`)
  * colour effects (`--colfx`)
  * annotation (`--annotate`, `--annotateex`)
  * dynamic range compression, or DRC (`--drc`)

* stereo (`--stereo`, `--decimate` and `--3dswap`), no support in libcamera for stereo currently.

* There is no image stabilisation (`--vstab`), it doesn't seem as though the legacy implementation actually does very much.

* There are no demo modes (`--demo`).

* The transformations supported are those that do not involve a transposition. 180 degree rotations, therefore, are among those permitted but 90 and 270 degree rotations are not.

* The metering modes supported are: centre, spot, average, matrix (average and matrix are actually the same) and custom (you would have to define a custom mode in the json tuning file).

* The exposure modes supported are: normal, sport, short (sport and short are synonyms). There is no "off" setting (it always rather baffled me as to what that meant).

* The supported AWB modes are: auto (also known as normal), incandescent, tungsten, fluorescent, indoor, daylight, cloudy and custom (for which you would have to define a custom mode in the json tuning file).
  * There is no setting for "off"; instead set the AWB gains (`--awbgains`) which disables the automatic algorithm.
  * libcamera does not have a "greyworld" AWB mode. To get this you will have to edit the "bayes" parameter in the json tuning file and set it to zero.

* There is support for setting the exposure time (`--shutter`) and analogue gain (`--analoggain` or just `--gain`). There is no explicit control of the digital gain; you get this if the gain requested is larger than the analogue gain can deliver by itself.

* libcamera has no understanding of ISO, so there is no `--ISO` option. Users should calculate the gain corresponding to the ISO value required (usually a manufacturer will tell you that, for example, a gain of 1 corresponds to an ISO of 40), and use the `--gain` parameter instead.

* There is no support for setting the flicker period yet.

* The following denoise modes are supported with the `--denoise` option:
  * `auto` - This is the default. Enabled spatial denoise. Use fast colour denoise for video/viewfinder, and high quality colour denoise for stills capture.
  * `off` - Disables spatial and colour denoise.
  * `cdn_off` - Disables colour denoise.
  * `cdn_fast` - Uses fast color denoise.
  * `cdn_hq` - Uses high quality colour denoise. Not appropriate for video/viewfinder due to reduced throughput.

* Per-frame data can be displayed on the titlebar with the `--info-text` command line argument with token substitution. Valid tokens are:
  * `%framenum` - frame number
  * `%fps` - framerate
  * `%exp` - Shutter speed
  * `%ag` - Analogue gain
  * `%dg` - Digital gain
  * `%rg` - Red colour gain
  * `%bg` - Blue colour gain
  * `%focus` - Focus figure of merit (FoM) value
  * `aelock` - 0 to indicate AE is converging, 1 to indicate AE is locked

#### libcamera-still

* raw output (`--raw`) is to a separate DNG file, not to the end of the jpeg.

* EXIF tags (`--exif`), libexif may not support all the exact same tags as proprietary GPU code.

* There is no burst mode (`--burst`). Much better performance is obtained by using libcamera-vid in MJPEG mode with `--segment 1` to create separate output files.

#### libcamera-vid

* MJPEG (`--codec mjpeg`) is controlled by a quality parameter like stills, not by a bitrate.

* The codec can be set to YUV420 (`--codec yuv420`) for uncompressed output (there is no libcamera-yuv app).

* Images cannot be displayed after the encoding process (`--penc`).

* Rate control does not support a fixed quantiser (`--qp`).

* No adaptive intra refresh options (`--irefresh`).

* Timed switching between pause/record is not supported (`--timed`).

* There is no access to vectors from the H.264 encoder (`--vectors` not supported).

* `--raw` and `--raw-format` are not supported, however, there is `--codec yuv420`.

#### libcamera-raw

* Writes bare raw sensor frames out into a single big file. There is no viewfinder image. If you want separate files, you can use `--segment 1` with the command.

Example Commands
----------------

Run these in your build directory.

```
./libcamera-hello -h
./libcamera-hello
./libcamera-hello --roi 0.25,0.25,0.5,0.5
./libcamera-hello --brightness 0.1 --contrast 1.1 --saturation 0.9 --sharpness 0.9

./libcamera-still -h
./libcamera-still -o test.jpg
./libcamera-still -r -o test.jpg
./libcamera-still -e png -o test.png
./libcamera-still -o test%04d.jpg -t 999999 --timelapse 10000
./libcamera-still --gain 2 -o test.jpg

./libcamera-vid -h
./libcamera-vid -t 10000 -o test.h264
./libcamera-vid -v -o test.mjpeg --codec mjpeg
./libcamera-vid -o test.h264 --inline --circular -k -t 0
./libcamera-vid -o segment%04d.h264 --inline --segment 5000 -t 100000
./libcamera-vid -o test.h264 --shutter 20000 --gain 1
./libcamera-vid -o test.h264 --framerate 15

./libcamera-raw -h
./libcamera-raw -o test.raw

./libcamera-jpeg -o test.jpg
```

Tips
----

* If you find yourself running out of memory, especially with the imx477 (HQ Cam) trying to stream very large frames, look in your `/boot/config.txt`. Replace `dtoverlay=vc4-fkms-v3d` by `dtoverlay=vc4-fkms-v3d,cma-512`. This will give you 512MB (a very large amount) of CMA memory.

* To try your own camera tuning, make a `raspberrypi` subfolder in a folder of your choice (let us say, `/home/pi`). Copy the JSON file for the camera to the `raspberrypi` subfolder, without changing its name, where you can edit it. Setting the following environment variable will cause libcamera to load your tuning file in preference, for example run `LIBCAMERA_IPA_CONFIG_PATH=/home/pi ./libcamera-hello`.

* When using the imx477 (HQ Cam) you can obtain the focus metric by running: `LIBCAMERA_LOG_LEVELS=RPiFocus:0 ./libcamera-hello -t 0`. It will be displayed in the terminal window (not on the image).

Known Issues
------------

* There is some screen tearing when using X/GLES for the preview window.
* Libcamera does not currently handle colour spaces, meaning that all videos are recorded in "full range BT601" YUV (as is the usual practice for JPEGs). This may be difficult to resolve until the necessary work has been done within libcamera.
* There appear to be some driver related issues lengthening the vblanking at startup under certain conditions, which can result in some initial AGC "wobble".
* ~~When exposure time is limited by framerate, the AGC can sometimes try to make up the difference using digital gain. A patch set for this is already in the works.~~
* ~~Low light images will tend to show more colour noise than images from the Broadcom stack. Work to correct this is in progress.~~
* No support yet for very long exposures.
* We don't yet have control of on-sensor DPC correction through the V4L2 drivers.
* Some of the camera tunings, especially sharpening for the v1 and v2 sensors, need to be re-visited.
* There is not currently a good way of forcing the HQ cam to select fast framerate modes. This needs support from the libcamera framework.