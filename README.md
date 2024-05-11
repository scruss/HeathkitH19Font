# HeathkitH19Font
an attempt to create an outline font of the Heathkit H19 terminal font, after [crt monitor - ZenithZ89 System Font - Retrocomputing Stack Exchange](https://retrocomputing.stackexchange.com/questions/29979/zenithz89-system-font)

## Process

From the ROM file *2716_444-29_h19font.bin* extracted from `h19.zip` at [MAME Heathkit H89 Emulator](https://heathkit.garlanger.com/emulator/mame/) it is fairly clear that 128 characters are encoded as blocks of 10 bytes out of every 16:

```
00000000: 1818 1818 1818 1818 1818 0000 0000 0000  ................ # char 0x00
00000010: 0000 0000 ffff 0000 0000 0000 0000 0000  ................ # char 0x01
00000020: 1818 1818 ffff 1818 1818 0000 0000 0000  ................ # char 0x02
 ...
00000200: 0000 0000 0000 0000 0000 0000 0000 0000  ................ # char 0x20, SPACE
00000210: 0008 0808 0808 0008 0000 0000 0000 0000  ................ # char 0x21, EXCLAMATION MARK
 ...
000007f0: 0000 1c3e 3e3e 1c00 0000 0000 0000 0000  ...>>>.......... # char 0x7f
```

The first 32 characters are graphic characters, and don't correspond to any currently used encoding.

The character data can be extracted and joined as contiguous data using:

```sh
for i in {0..127}
do
  dd if=2716_444-29_h19font.bin bs=1 skip="$((i * 16))" count=10 of="$(printf 'h19-%02x.bin' ${i})" status=none
done
cat h19-*.bin > raw-h19.bin
```

John Elliott's [PSF Tools](https://www.seasip.info/Unix/PSF/index.html) can be used to convert this raw binary file into slightly more useful standard bitmap font files:

```sh
raw2psf --height=10 --width=8 --first=0 --last=127 raw-h19.bin h19.psf
psf2bdf --first=0 --last=127 --fontname=HeathkitH19 --descent=2 --defchar=9 h19.psf HeathkitH19.bdf
```

Note that the [Bitmap Distribution Format (BDF)](https://en.wikipedia.org/wiki/Glyph_Bitmap_Distribution_Format) file created is effectively un-encoded. It does, however, contain all of the bitmap characters.

[bdf2sfd](https://github.com/fcambus/bdf2sfd) did the heavy lifting of converting from a bitmap to an initial traced outline:
```sh
bdf2sfd -f 'Heathkit-H19 Regular' HeathkitH19.bdf > Heathkit-H19-Regular.sfd
```

### Re-encoding

The non-ASCII characters were re-encoded using FontForge like so:

```
U+00B1	±	PLUS-MINUS SIGN
U+00B6	¶	PILCROW SIGN
U+00F7	÷	DIVISION SIGN
U+2022	•	BULLET
U+2192	→	RIGHTWARDS ARROW
U+2193	↓	DOWNWARDS ARROW
U+2501	━	BOX DRAWINGS HEAVY HORIZONTAL
U+2503	┃	BOX DRAWINGS HEAVY VERTICAL
U+250F	┏	BOX DRAWINGS HEAVY DOWN AND RIGHT
U+2513	┓	BOX DRAWINGS HEAVY DOWN AND LEFT
U+2517	┗	BOX DRAWINGS HEAVY UP AND RIGHT
U+251B	┛	BOX DRAWINGS HEAVY UP AND LEFT
U+2523	┣	BOX DRAWINGS HEAVY VERTICAL AND RIGHT
U+252B	┫	BOX DRAWINGS HEAVY VERTICAL AND LEFT
U+2533	┳	BOX DRAWINGS HEAVY DOWN AND HORIZONTAL
U+253B	┻	BOX DRAWINGS HEAVY UP AND HORIZONTAL
U+254B	╋	BOX DRAWINGS HEAVY VERTICAL AND HORIZONTAL
U+2571	╱	BOX DRAWINGS LIGHT DIAGONAL UPPER RIGHT TO LOWER LEFT
U+2572	╲	BOX DRAWINGS LIGHT DIAGONAL UPPER LEFT TO LOWER RIGHT
U+2573	╳	BOX DRAWINGS LIGHT DIAGONAL CROSS
U+2580	▀	UPPER HALF BLOCK
U+2582	▂	LOWER ONE QUARTER BLOCK
U+258E	▎	LEFT ONE QUARTER BLOCK
U+2590	▐	RIGHT HALF BLOCK
U+2592	▒	MEDIUM SHADE
U+2596	▖	QUADRANT LOWER LEFT
U+2597	▗	QUADRANT LOWER RIGHT
U+2598	▘	QUADRANT UPPER LEFT
U+259D	▝	QUADRANT UPPER RIGHT
U+25E4	◤	BLACK UPPER LEFT TRIANGLE
U+25E5	◥	BLACK UPPER RIGHT TRIANGLE
U+1FB82	🮂	UPPER ONE QUARTER BLOCK
U+1FB87	🮇	RIGHT ONE QUARTER BLOCK
```

Note that the upper and lower quarter block mapping is not quite 100% accurate. The H19 font is 10 px high, but there is no *ONE FIFTH BLOCK* in Unicode (yet).

## TBD

* clean up the inevitable mess this process made.

## Licence

 <p xmlns:cc="http://creativecommons.org/ns#" xmlns:dct="http://purl.org/dc/terms/"><a property="dct:title" rel="cc:attributionURL" href="https://github.com/scruss/HeathkitH19Font">Heathkit H19 Terminal Font</a> by <a rel="cc:attributionURL dct:creator" property="cc:attributionName" href="https://scruss.com/blog/">Stewart Russell</a> is licensed under <a href="https://creativecommons.org/licenses/by-sa/4.0/?ref=chooser-v1" target="_blank" rel="license noopener noreferrer" style="display:inline-block;">CC BY-SA 4.0<img style="height:22px!important;margin-left:3px;vertical-align:text-bottom;" src="https://mirrors.creativecommons.org/presskit/icons/cc.svg?ref=chooser-v1" alt=""><img style="height:22px!important;margin-left:3px;vertical-align:text-bottom;" src="https://mirrors.creativecommons.org/presskit/icons/by.svg?ref=chooser-v1" alt=""><img style="height:22px!important;margin-left:3px;vertical-align:text-bottom;" src="https://mirrors.creativecommons.org/presskit/icons/sa.svg?ref=chooser-v1" alt=""></a></p> 
