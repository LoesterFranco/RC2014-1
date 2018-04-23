# Hexload

Hexload is a basic program, that uploads an assembly language program to support uploading other programs into the memory of the RC2014.

## C and Assembly file upload for new users (No EEPROM programmer required)

Just built a Classic or Mini RC2014, and don't yet have an EEPROM burner?

Do you want to do Assembly or C language programming, and then load and run these programs on your RC2014?

<div>
<table style="border: 2px solid #cccccc;">
<tbody>
<tr>
<td style="border: 1px solid #cccccc; padding: 6px;"><a href="http://rc2014.co.uk/wp-content/uploads/2014/10/RC2014_Overview.jpg" target="_blank"><img src="http://rc2014.co.uk/wp-content/uploads/2014/10/RC2014_Overview.jpg"/></a></td>
</tr>
<tr>
<th style="border: 1px solid #cccccc; padding: 6px;"><centre>RC2014 Classic<center></th>
</tr>
<tr>
<td style="border: 1px solid #cccccc; padding: 6px;"><a href="http://rc2014.co.uk/wp-content/uploads/2016/10/2016-10-04t16-58-14.434z-mini2.jpg.855x570_q85_pad_rcrop01.jpg-1.jpeg" target="_blank"><img src="http://rc2014.co.uk/wp-content/uploads/2016/10/2016-10-04t16-58-14.434z-mini2.jpg.855x570_q85_pad_rcrop01.jpg-1.jpeg"/></a></td>
</tr>
<tr>
<th style="border: 1px solid #cccccc; padding: 6px;"><centre>RC2014 Mini<center></th>
</tr>
</tbody>
</table>
</div>

`hexload` is the way you can upload and run either assembly or C compiled code on your RC2014.

- Connect to the RC2014 with a serial interface
- Press the Reset button Boot the machine to restart the RC2014
- Select Cold start and set Memory Top? to 35071.
- Set your UART interface program (like minicom or Putty) so that it waits at least 10 msec between each byte and copy-paste the file [`hexload.bas`](https://raw.githubusercontent.com/RC2014Z80/RC2014/master/BASIC-Programs/hexload/hexload.bas) or use `slowprint.py` as directed below.

You should see one of these two messages.

```bash
80 SBC By Grant Searle

Memory top? 35071
Z80 BASIC Ver 4.7b
Copyright (C) 1978 by Microsoft
1916 Bytes free
Ok
```

or

```bash
Z80 SBC By Grant Searle

Cold or warm start (C or W)? C

Memory top? 35071
Z80 BASIC Ver 4.7b
Copyright (C) 1978 by Microsoft
1916 Bytes free
Ok
```

Once you have pasted all of the `hexload.bas` file into your terminal,
the following message should appear (following lots of Basic commands):

```bash
...
...
run
Loading Data
Start Address: 8900
End Address:   89D7
USR(0) -> HexLoad
HEX LOADER by Filippo Bergamasco & feilipu for z88dk

:
```

At that point, copy-paste the desired Intel HEX file, i.e. the Assembly or C language compiled program in HEX format, into the terminal.

An example `helloworld.hex` is provided to check that this process is working. You should see the "RC2014" hello world as below.

```bash
...
...
9550 data 999
9999 END
run
Loading Data
Start Address: 8900
End Address:   89D7
USR(0) -> HexLoad
HEX LOADER by Filippo Bergamasco & feilipu for z88dk

:###############################################################################
################################################################################
################################################################################
#####

Done

 0 
USR(0) -> 0x9000, z88dk default

Hello World R
Hello World C
Hello World 2
Hello World 0
Hello World 1
Hello World 4

 0 
Ok
```

## Program upload from a Linux system

On a Linux system assuming an USB-to-serial converter attached as `/dev/ttyUSB0` the following
commands can be used:

```bash
$ cd hexload
$ python slowprint.py < hexload.bas > /dev/ttyUSB0
$ cat < helloworld.hex > /dev/ttyUSB0
```

## How does `hexload` work?

So how does this process work?

The RC2014 Classic or Mini has 32kB of RAM, that is located from address `0x8000`. The MS Basic program installed by default uses some bytes from `0x8000` through to `0x8183` to manage itself, and it usually leaves the bytes from there through to `0xFFFF` free for BASIC programs.

The MS Basic language has a special command `USR(x)` which can cause Basic to jump to a user program, when the address of that program is written to a special location. For the default ROM this location is the two bytes from `0x8049`. If we write the address of our program to that location, then when we call the Basic `USR(0)` program, then the RC2014 will begin executing our program.

So with `hexload` we do this twice. Firstly, to run the actual Intel HEX upload program from address `0x8900` after we upload it, and then again to run our own program to the location from `0x9000` after it is uploaded.

We have to let MS Basic know not to use this space for its programs, hence when we cold start the RC2014 we ask it to keep the top memory address to 35071, or in hexadecimal `0x88FF`, just below the intended location for our `hexload` program.

Before we can run the `hexload` program we have to upload and run a Basic program to `poke` the program into the right location, from `0x8900` to `0x89D7`, set the starting address for `USR(0)` correctly, and then set it running. And this program has to fit within the 1916 bytes available to Basic.

Once we have the `hexload` colon prompt then we can use it to upload, and also start, our own program with the origin of `0x9000`.

```bash
Loading Data
Start Address: 8900
End Address:   89D7
USR(0) -> HexLoad
HEX LOADER by Filippo Bergamasco & feilipu for z88dk

:
```

An additional test program provided by Z88DK is the `password.hex` program also in this directory, which demonstrates the terminal editing capabilities of the Z88DK standard library. It is available in the examples directory of Z88dk.


## How to prepare C Programs?

The easiest way to compile C programs for the RC2014 is using the Z88DK.

The simple command line (to get started) looks like this below.

```bash
zcc +rc2014 -subtype=basic -clib=sdcc_iy helloworld.c -o helloworld -create-app
```

Whilst there are many additional options which can add information about the process, the above provides the intel hex or `ihx` information that can be uploaded to the RC2014 and run.

The `+rc2014` advises that the machine is the RC2014, and the `-subtype=basic` advises that the program should be compiled with `0x9000` as its origin, and that it should use the serial drivers included in the MS Basic ROM.

The [Z88DK examples](https://github.com/z88dk/z88dk/tree/master/libsrc/_DEVELOPMENT/EXAMPLES) include some classic games including StarTrek, Sudoku, Eliza, and Chess, all written in C, that can be compiled and run on the RC2014.

The [Z88DK source code](https://github.com/z88dk/z88dk) has a (actually two) full standard C libraries, and two alternative compilers (sccz80 and patched sdcc) to chose from. It supports over 50 machine types (including the RC2014), and is very actively maintained. But initially, the above incantation provides a good result.

Full instructions to use the Z88DK are available from the [Wiki](https://www.z88dk.org/wiki/doku.php) and support is available on the [forum](https://www.z88dk.org/forum/forums.php).