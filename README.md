## BlackMagic devices tools

Initially developed as an example integration between libavformat and the
bmd sdk, it ended up being a set of useful tools to use the BlackMagic Devices
decklink cards on Linux and OSX.

Thanks to TodoStreaming sponsoring its early development.

## Build instructions

In order to build it just clone/unpack this on your Sample directory from the
DeckLink SDK and then issue "make". If you have [Libav][1] and [pkg-config][2] or
[pkgconf][3] installed it will build fine.

    Make sure you are using at least Libav10 otherwise it will not build.

You can build it out of the Sample tree by issuing

```sh
make SDK_PATH=/path/to/the/bmd/include
```

### OSX Support

Should work out of box.

### Windows Support

The tools do not build on Windows currently, supporting it would either
require a working widl support in mingw64 or access to the native tools.

Patch and/or sponsorship welcome.

## Usage

Provide a consistent UDP stream with automatic resolution changes:

```sh
./bmdcapture -C 0 -V 3 -F nut -f pipe:1 | ffmpeg -i - -c:v h264 -c:a mp3 -r 24 -g 24 -rtbufsize 2048M -threads 2 -filter:v "scale=iw*min(1920/iw\,1080/ih):ih*min(1920/iw\,1080/ih), pad=1920:1080:(1920-iw*min(1920/iw\,1080/ih))/2:(1080-ih*min(1920/iw\,1080/ih))/2" -mpegts_flags resend_headers -pix_fmt yuv420p -f mpegts udp://localhost:4000
```

To work this should be combined with the following systemd unit

```
[Unit]
Description=Blackmagic video streaming 4000
Requires=network-online.target
After=network-online.target

[Service]
EnvironmentFile=/home/aca-apps/config/env
PIDFile=/var/run/capture0.pid
ExecStartPre=/bin/rm -f /var/run/capture0.pid
ExecStart=/bin/bash -c '/home/aca-apps/bmdtools/bmdcapture -C 0 -V 3 -F nut -f pipe:1 | ffmpeg -i - -c:v h264 -c:a mp3 -r 24 -g 24 -rtbufsize 2048M -threads 2 -filter:v "scale=iw*min(1920/iw\,1080/ih):ih*min(1920/iw\,1080/ih), pad=1920:1080:(1920-iw*min(1920/iw\,1080/ih))/2:(1080-ih*min(1920/iw\,1080/ih))/2" -mpegts_flags resend_headers -pix_fmt yuv420p -f mpegts udp://localhost:4000 & echo $! >> /var/run/capture0.pid'
Restart=always

[Install]
WantedBy=multi-user.target

# Require Blackmagic drivers and FFmpeg to be installed
```


```sh
./bmdcapture -C 1 -m 2 -F nut -o strict=experimental:syncpoints=none -f pipe:1 | avconv -vsync passthrough -y -i - <your options here>
```

-C select the capture device if more than one is present.

-F define the container format, I suggest using nut.

-f output file name, any libavformat compatible url is supported.

-m specific modeline, resolution+framerate

-o pass AVFormat AVOptions (expert)

> NOTE: make sure you are processing frames capture in real time or be
prepared to end up using all your memory quite quickly, HD raw data
fills up memory quickly.

```sh
avconv -vsync 1 -i <source> -c:v rawvideo -pix_fmt uyvy422 -c:a pcm_s16le -ar 48000 -f nut -f_strict experimental -syncpoints none - | ./bmdplay -f pipe:0
```

> NOTE: The default NUT syncpoint strategy uses additional memory and could
consume more memory than expected.


## Support

The github [issue tracker](https://github.com/lu-zero/bmdtools/issues) can
be used to track bugs and feature requests.

### Contact

You can directly contact me either at

lu_zero@gentoo.org or luca.barbato@luminem.it

### Paid Support

Paid support is offered, contact info@luminem.it for details.

[1]: http://libav.org
[2]: http://www.freedesktop.org/wiki/Software/pkg-config/
[3]: https://github.com/pkgconf/pkgconf
