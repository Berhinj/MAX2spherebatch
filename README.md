# MAX2Sphere Batch

This is a enhanced version of [MAX2Sphere](https://github.com/trek-view/max2sphere), with some improvements:
- Handling batch input (improvement provided by the author trek-view but recently abandonned - still, all credits goes to him)
- Exporting cube maps with the batch approach (improvement provided by BerhinJ)

MAX2Sphere Batch takes raw GoPro .360 frames (two strips encoded as EAC) and converts them to a more widely recognised equirectangular and cube map.

## Handling batch input

MAX2Sphere was designed for single inputs. Our use-case called for conversion of multiple frames (extracted from videos).

The problem with MAX2Sphere is it loaded the settings for each frame with each one processed (which was wasteful).

MAX2Sphere uses lookup tables that are called once for each run.

### How lookup tables are handled

The process in the same for both cube map and equirectangular (each process has its own lookup tables).

MAX2spherebatch will first look for a lookup table.

If it finds one it will read it and use it during the current processing run.

If it doesn't find a lookup table (or if the read above fails) it will create one and save it to disk and then use it during the current processing run.

Lookup tables rely on four values: the template number, the output width and height, and the antialising value.

e.g. Template 0, width 5376, height 2688 and antialising of 2.

In the case above where the output image width was autodetermined the lookup table is called `0_5376_2688_2.data`

The template refers to the recording mode, the template defines the various geometric values the code needs in order to extract out the parts correctly.

Currently two recording modes are supported. At the top of the MAX2spherebatch.c file you will see the line referencing them:

```
FRAMESPECS template[NTEMPLATE] = {{4096,1344,1376,1344,32,5376},{2272,736,768,736,16,2944}};
```

The lookup table does take almost no time to read, compared to calculating the lookup table. However it does end up taking a decent about of disk space, almost 700MB in this case.


## Exporting cube maps


## Installation

The MAX2sphere command line utility should build out of the box on Linux using the simple Makefile provided. The only external dependency is the standard jpeg library (libjpeg), the lib and include directories need to be on the gcc build path. The same applies to MacOS except Xcode and command line tools need to be installed.

```
$ git clone https://github.com/BerhinJ/MAX2spherebatch
$ make -f Makefile
$ @SYSTEM_PATH/max2spherebatch
```

## Usage

### Preparation

This script is designed to be used with frames. You will need to first convert a `.360` video to frames and then pass the two corresponding frames to the script.

You can use ffmpeg to split your `.360` video into frames. These parameters will split the video into frames respectively to the shutter speed (which is ideal to then join it with the telemetry metadata), minimize compression and leverages multithreading (8).

```
$ ffmpeg -i INPUT.360 -loglevel error -vsync vfr -threads 8 -map 0:0 -q:v 0 track0/img%d.jpg -map 0:5 -q:v 0 track5/img%d.jpg
```

### Script

The sequence filename template should contain two `%d` entries. The first will be populated with the track number 0 or 5, the second is the frame sequence number, see `-n` and `-m` below.

So for example, if there are 1000 frames called track0_frame0001.jpg, track5_0001.jpg, ... then the program might be called as follows:

```
$ @SYSTEM_PATH/MAX2spherebatch -w 5760 -n 1 -m 1000 track%d_frame%04d.jpg
```

Or if directories are used with frames track0/frame1.jpg, track5/1000.jpg, ...

```
$ @SYSTEM_PATH/MAX2spherebatch -w 5760 -n 1 -m 1000 track%d/frame%4d.jpg
```

Options:

* `-w` n sets the output image width, default: -1
* `-a` n sets antialiasing level, default = 2
* `-o` s specify the output filename, default is based on track0 name. If specified then it should contain one `%d` field for the frame number
* `-n` n Start index for the sequence, default: 0
* `-m` n End index for the sequence, default: 100000
* `-d` enable debug mode, default: off

#### Examples (MacOS)

##### Use a GoPro Max 5.6K video (width = 5760)

```
$ @SYSTEM_PATH/max2spherebatch -w 5760 -n 1 -m 10 -o testframes/5-6k/GS018421_%d.jpg testframes/5-6k/track%d/GS018421_%d.jpg
```

### Metadata

Note, the resulting image frames will not have any metadata -- this is not covered by the script.

### Camera support

This script has currently been tested with the following cameras and modes:

* GoPro MAX
	* 360 Video (output .360)
		* 5.6K stitched (24FPS, 30FPS)
		* 3k stitched (60 FPS)
	* 360 TimeWarp (output .360)
		* (2x,5x,10x,15x,30x frame rate)

## Debugging

#### Failed to open warning

If you see the warning:

```
ReadFrame() - Failed to open "testframes/5-6k/track0/GS018421_11.jpg"
```

It is likely the value for `-m` used does not match the number of frames in the directories used. This is not a critical error, and you should find all processed frames.

## License

The code of this site is licensed under an [MIT License](/LICENSE).

## Credits

Again, major credit to [trek-view](https://github.com/trek-view), the intial author of MAX2spherebatch, even he ended up removing his script, I only extended his script for cube map (plus some minor fixes and cleanings).