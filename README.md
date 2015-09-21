wav2dvdlpcm-24bit by John Regan, 2009

This code is under a BSD license, as it's really pretty trivial.

If you do use it in a project, though, let me know!
saxindustries - AT - gmail - DOT - com

This program reads in a 24-bit, signed, little-endian wav or raw audio file,
and spits out a headerless file, suitable for use on DVD.  It all occurs
on stdin and stdout, making it suitable for piping audio from some
other process.

It doesn't do any kind of checking whatsoever on the file.  It doesn't
check if you're piping in anything at all.
You're responsible for providing the correct audio format, as well as
making sure that you're piping in at least something.

It accepts one option - whether or not you're piping raw audio.
This option is declared with -r, for raw.

There's no makefile, you could compile it with something like:
`gcc -o wav2dvdlpcm-24bit wav2dvdlpcm-24bit.c`

Then execute it with something like:
`wav2dvdlpcm-24bit <infile.wav >outfile.lpcm`

Or, from a raw-audio file:
`wav2dvdlpcm-24bit -r <infile.raw >outfile.lpcm`

Or, from a flac file:
`flac -d -c infile.flac | wav2dvdlpcm-24bit >outfile.lpcm`

Have fun!