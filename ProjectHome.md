I was really frustrated with the state of decoding 24-bit, 96kHz FLAC audio to something I could burn onto a standard Video-DVD.  There are some tools that accomplish this, but they were all too all-in-one for my taste, so I wrote this short program.

This program reads in a 24-bit, signed, little-endian wav or raw audio file, and spits out a headerless file, suitable for use on DVD.  It all occurs on stdin and stdout, making it suitable for piping audio from some other process.

Again, all it produces is an LPCM stream.  You'll need to feed it into mplex, dvdauthor, etc, on your own.