# Introduction #

This is a brief example of using this program, along with mjpegtools, imagemagick, flac, cdrkit, and dvdauthor to genereate a DVD image, with each song track being a chapter.

THIS GUIDE IS A WORK IN PROGRESS AND IS NOT YET FINISHED


# Details #

For this example, I'm going to be using the 24-bit, 96kHz FLAC version of Nine Inch Nails' [The Slip](http://theslip.nin.com/).  It's available for free, so it makes for a great example.

I'm assuming that, at this point, you've already obtained the material, and have all needed programs installed, including wav2dvdlpcm-24bit.

## Step 1 ##
Navigate to your project directory, we should have all the FLAC files in here.  We'll setup an empty audio.lpcm file, as well as an empty start-points file, and an end-point file:

```
touch audio.lpcm
touch start-points.txt
touch end-point.txt
```

The start-points value will contain our track starting points, as raw audio bytes.  We'll use these to figure out our chapters.

## Step 2 - Generate Audio Stream ##
Next, we'll decode the audio, and keep track of where each track starts, as well as the final byte count:
```
for file in *.flac; do stat -c "%s" audio.lpcm >> start-points.txt; flac -d -c --force-raw-format --sign=signed --endian=little "$file" | wav2dvdlpcm-24bit -r >> audio.lpcm; done; stat -c "%s" audio.lpcm >> end-point.txt
```

So now, assuming you're using the Nine Inch Nails "The Slip," your start-points file should contain:
```
0
49282704
185338860
317531808
466821636
631124508
801110796
922043748
1183340856
1343253876
```
and the end-point should be:
```
1514958180
```

## Step 3 - Generate Video Stream ##
Next, let's calculate the total number of frames we're going to need, and encode a basic video with that number of frames:
```
framerate=24000/1001; read size < end-point.txt ; frames=`echo "scale=10;($size/576000)*($framerate)" | bc | sed 's/[.].*//' `; convert xc:black -scale 352x240! -depth 8 bg.ppm; ppmtoy4m -n $frames -F 24000:1001 -A 10:11 -I p -r -S 420mpeg2 bg.ppm | mpeg2enc -f 8 -p -M 4 -b 200 -o video.m2v
```

## Step 4 - Multiplex and Author ##
Now, we can multiplex:
```
mplex -f 8 -o dvd-ready.mpg video.m2v audio.lpcm -L96000:2:24
```

The next step is calculating the chapter points, and running this whole thing through dvdauthor.
```
chapters=""; for point in `cat start-points.txt`; do fseconds=`echo "scale=10;($point/576000)" | bc` ; seconds=` echo $fseconds | sed 's/[.].*//'`; hours=$((seconds/3600)); seconds=$((seconds%3600)); minutes=$((seconds/60)); seconds=$((seconds%60)); afseconds=`echo $fseconds | sed 's/.*[.]//'`; chapters=`printf "%s%02d:%02d:%02.3f," "$chapters" $hours $minutes "$seconds.$afseconds"`; done; chapters=${chapters%","}; dvdauthor -o THE_SLIP -t -c "$chapters" --file=dvd-ready.mpg
dvdauthor -o THE_SLIP -T
```

# This whole thing as a bash script #
Here's a slightly modified version of the above, as a handy and awesome bash script.  I just cd to the directory containing FLAC files, and run the script:
```
#!/bin/bash

#Setup the frame size, the frame rate, aspect ratio, whether we want pulldown
framesize=352x240
framerate=24000/1001
mpeg2enc_framerate=24000:1001
aspect=10:11
pulldown=-p

#the audio byterate = channels * (bits/8) * frequency
#so, for 24-bit, 96kHz: 2 * (24/8) * 96000 = 576000
byterate=576000

#specify the output directory
outdir=THE_SLIP

touch audio.lpcm
touch start-points.txt

for file in *.flac
do
  stat -c "%s" audio.lpcm >> start-points.txt
  flac -d -c --force-raw-format --sign=signed --endian=little "$file" | \
  wav2dvdlpcm-24bit -r >> audio.lpcm
done

audiosize=`stat -c "%s" audio.lpcm`

frames=`echo "scale=10;($audiosize/$byterate)*($framerate)" | bc | sed 's/[.].*//'`

convert xc:black -scale $framesize! -depth 8 bg.ppm

ppmtoy4m -n $frames -F $mpeg2enc_framerate -A $aspect -I p -r -S 420mpeg2 bg.ppm |\
mpeg2enc -f 8 $pulldown -b 200 -o video.m2v

rm bg.ppm

mplex -f 8 -o dvd-ready.mpg video.m2v audio.lpcm -L96000:2:24

rm audio.lpcm
rm video.m2v

chapters=""
for point in `cat start-points.txt`
do
  #fseconds = raw number of seconds
  fseconds=`echo "scale=10;($point/$byterate)" | bc`

  #seconds will be our seconds *without* the decimal
  seconds=`echo $fseconds | sed 's/[.].*//'`

  hours=$((seconds/3600))
  seconds=$((seconds%3600))
  minutes=$((seconds/60))
  seconds=$((seconds%60))

  #afseconds will be everything after the decimal
  afseconds=`echo $fseconds | sed 's/.*[.]//'`

  chapters=`printf "%s%02d:%02d:%02.3f," \
  "$chapters" $hours $minutes "$seconds.$afseconds"`
done

chapters=${chapters%","}

dvdauthor -o $outdir -t -c "$chapters" --file=dvd-ready.mpg
dvdauthor -o $outdir -T

rm dvd-ready.mpg
rm start-points.txt

mkisofs -dvd-video -udf -V "$outdir" -o "$outdir.iso" "$outdir"

rm -rf $outdir
```