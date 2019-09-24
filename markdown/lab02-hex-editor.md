---
title: Hex Editor
...

# Introduction

Whether punch cards, paper tape, magnetic tape, magnetic disk, optical disc, NAND-flash, or technologies yet unknown,
large quantities of digital information have always been, and likely will always remain, giant streams of sequential bits.
These are stored physically in something, so they have limited flexibility:
you can change a bit to 0 or 1, but you can't remove a bit entirely without moving every other bit over to fill in the gap.

When interacting with such "binary files," it is typical to use a tool known as a "hex editor."
This shows each byte (set of eight bits) as a two-digit hexadecimal value
and allows users to edit that information in place.

In this lab, you'll explore some of what hex editors can do.

# Getting a hex editor

Because we are having setup issues with the department servers, we'll use an online hex editor:

<https://hexed.it/>

Although it is an online tool, you need to open files from your local drive;
your basic process is

1. Download a file to edit
2. Go to [hexed.it](https://hexed.it){target="_blank"}
3. Use the "Open file" option on the top to load the file
4. Edit the file as desired
5. Use "Export" to save the file 


# ASCII

Many files contain textual information, generally encoded using ASCII or a compatible superset of that such as UTF-8, ISO-8859-1, Windows-1252, Mac-Roman, etc.
In these, each character is mapped to a single byte between 0 and 127; bytes larger than 127 are often parts of multi-byte character encodings and vary by encoding variant.

<div style="columns: 3">

	Hex 	Char
----------  --------------
 	09      TAB (`\t`)
	0A 		LF  (`\n`)
	0D 		CR  (`\r`)
	20 	    Space
	21 	    `!`
	22 	    `"`
	23 	    `#`
	24 	    `$`
	25 	    `%`
	26 	    `&`
	27 	    `'`
	28 	    `(`
	29 	    `)`
	2A 	    `*`
	2B 	    `+`
	2C 	    `,`
	2D 	    `-`
	2E 	    `.`
	2F 	    `/`
	30 	    `0`
	31 	    `1`
	32 	    `2`
	33 	    `3`
	34 	    `4`
	35 	    `5`
	36 	    `6`
	37 	    `7`
	38 	    `8`
	39 	    `9`
	3A 	    `:`
	3B 	    `;`
	3C 	    `<`
	3D 	    `=`
	3E 	    `>`
	3F 	    `?`

	Hex 	Char
----------  --------------
	40 	    `@`
	41 	    `A`
	42 	    `B`
	43 	    `C`
	44 	    `D`
	45 	    `E`
	46 	    `F`
	47 	    `G`
	48 	    `H`
	49 	    `I`
	4A 	    `J`
	4B 	    `K`
	4C 	    `L`
	4D 	    `M`
	4E 	    `N`
	4F 	    `O`
	50 	    `P`
	51 	    `Q`
	52 	    `R`
	53 	    `S`
	54 	    `T`
	55 	    `U`
	56 	    `V`
	57 	    `W`
	58 	    `X`
	59 	    `Y`
	5A 	    `Z`
	5B 	    `[`
	5C 	    `\`
	5D 	    `]`
	5E 	    `^`
	5F 	    `_`

	Hex 	Char
----------  --------------
	60 	    <code>\`</code>
	61 	    `a`
	62 	    `b`
	63 	    `c`
 	64 	    `d`
 	65 	    `e`
 	66 	    `f`
 	67 	    `g`
 	68 	    `h`
 	69 	    `i`
 	6A 	    `j`
 	6B 	    `k`
 	6C 	    `l`
 	6D 	    `m`
 	6E 	    `n`
 	6F 	    `o`
 	70 	    `p`
 	71 	    `q`
 	72 	    `r`
 	73 	    `s`
 	74 	    `t`
 	75 	    `u`
 	76 	    `v`
 	77 	    `w`
 	78 	    `x`
 	79 	    `y`
 	7A 	    `z`
 	7B 	    `{`
 	7C 	    `|`
 	7D 	    `}`
 	7E 	    `~`

</div>

Download [ritchie.txt](files/ritchie.txt) and open it in your hex editor.
You should see both a hex edit space and a character edit space, along with some numerical interpretations.

To check off this lab, you'll need to demonstrate to your TA that you can do the following *in the hex editing space*:

1. Put the first line into all upper-case.
2. Replace all line breaks with spaces.

# Images

There are many image formats, and many of them use esoteric math to encode large regions of color succinctly.
However, some are uncompressed and represent colors directly;
these generally include a header of some kind, followed by three bytes per pixel:
a red byte (0 for no red light, 255 for right red light), a green byte (also 0--255), and a blue byte (ditto).

One such uncompressed format is the [BMP file format](https://en.wikipedia.org/wiki/BMP_file_format).
The BMP file format hgas several variants, but the most common has

- Width of image as a little-endian 32-bit number in bytes 18--21
- Height of image as a little-endian 32-bit number in bytes 22--25
- Image data starting at a byte whose index is given in a 4-byte little-endian number in bytes 10--13

![low-res Dennis Ritchie headshot](files/Dennis-Ritchie.bmp)

Download [Dennis-Ritchie.bmp](files/Dennis-Ritchie.bmp) and do the following:

1. Swap the width and height values and save the result as `swap.bmp`. This should look odd and horizontally streaky.

2. Steganography is the art of hiding information such that people seeing it don't know it is even there.
    One simple form of steganography hides information in the low-order bits of each pixel in an image.
    
    Starting from the original [Dennis-Ritchie.bmp](files/Dennis-Ritchie.bmp), encode "win" in the low-order bit of the first 24 bytes of image data.
    In particular,
    
    1. convert "win" to 3 bytes using ASCII
    2. Encode those 3 bytes as 24 big-endian bits
    3. For each of the first 24 image data bytes
        - set the low-order bit of the byte to 0 if the corresponding bit of "win" is 0
        - set the low-order bit of the byte to 1 if the corresponding bit of "win" is 1
        
        {.example ...}
        Suppose we want to encode "x" in the low-order bits of data consisting of bytes `01 23 54 76 89 a4 cd 5f`.
        Since "x" is ASCII 78~16~, or 01111000~2~, we encode this by setting the lower-order bits as follows:
        
        -----------	--------------	-----------------
        `01`		set bit to 0	`00` (changed bit)
        `23`		set bit to 1	`23` (unchanged, as it was already 1)
        `54`		set bit to 1	`55` (changed bit)
        `76`		set bit to 1	`77` (changed bit)
        `89`		set bit to 1	`89` (unchanged, as it was already 1)
        `a4`		set bit to 0	`a4` (unchanged, as it was already 0)
        `cd`		set bit to 0	`cc` (changed bit)
        `5f`		set bit to 0	`5e` (changed bit)
        -----------	--------------	-----------------

        {/}
        
    4. Save the resulting image as `hide.bmp` and verify that it does not look odd when viewed as an image


# Check-off

To check-off this lab, show a TA 

- show your TA you can replace newlines and capitalize words on the hex side of the editor, and describe the pattern you found doing so
- show your TA both your new images, and let them see the encoded data in the hex editor for `hide.bmp`

For most students, this should happen in lab;
if you have completed the lab exercise before lab occurs, you are welcome to do it in their office hours.

