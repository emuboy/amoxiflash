# Introduction #

The Infectus chip communicates with the host system using a fairly simple USB protocol carried over two bulk endpoints -- one in each direction.

Commands are sent in small binary packets.  The first byte indicates the category of command; the second byte indicates the actual command, and the rest of the bytes carry any parameters.  Every command is at least 8 bytes long, and padded with zeroes as necessary.  The length and parameter format must be determined by looking at the first two bytes.

All responses seem to begin with the byte 0xFF -- this is probably a return value.

# Initialization  - 0x45 #
0x45 commands seem to be handled by the SiLabs MCU onboard the Infectus chip.

```
 45 15 00 00 00 00 00 00     -- reset chip
 45 13 01 00 00 00 00 00     -- get chip revision?  returns 0x82 for me
 45 14 xx 00 00 00 00 00     -- select NAND bank -- xx should be either 00 or 01, depending on which chip in a dual-chip configuration should be used.   
                                Even if only a single NAND chip is present, this command must be sent to initialize the chip.
```

# PLD Info queries - 0x4c #
0x4C commands retrieve info about the programing of the chip.

```
 4c 07 00 00 00 00 00 00     -- get "loader version" number
 4c 15 00 00 00 00 00 00     -- get "PLD ID" -- this identifies which "Actel program" has been loaded onto the device
```

# NAND commands - 0x4e 0x00 #
The Infectus software sends raw NAND commands to the chip.  This is a good thing, because it means we should be able to add support for any NAND flash chip that we can obtain a datasheet for.

Note: the below commands are just meant as examples.  They are correct for the 512MB large-block Wii flash chips I've tried, but will vary between chips; consult your local datasheet for more info.

```
4e 00 00 00 00 00 00 xx yy zz ... xx is the number of parameters
                                  yy is the command ID
                                  zz ... are parameters

4e 00 00 00 00 00 00 00 FF    -- reset flash

4e 00 00 00 00 00 00          -- read flash ID
   01 90 00                   

4e 00 00 00 00 00 00 00 70    -- read flash status

4e 00 00 00 00 00 00          -- erase PRE
   03 60 {p0 p1 p2}              {p2,p1,p0} is 24-bit page number (so, send page 0x40 to erase block 1)

4e 00 00 00 00 00 00 00 d0    -- erase confirm

4e 00 00 00 00 00 00          -- page read PRE
   05 00 {o0 o1} {p0 p1 p2}      {p2, p1, p0} is 24-bit page number
                                 {o1, o0} is offset within the block (usually 0)

4e 00 00 00 00 00 00 00 30    -- page read confirm

4e 00 00 00 00 00 00          -- page program PRE
   05 80 {o0 o1} {p0 p1 p2}      {p2, p1, p0} is 24-bit page number
                                 {o1, o0} is offset within the block (see discussion below)

4e 00 00 00 00 00 00 00 10    -- page program confirm

 
```

# Data Transfers - 0x4e 0x02 / 0x4e 0x01 #
The Infectus chip seems to have a small buffer which is used to support bulk data transfers between the chip and the NAND flash part, to be used when reading or programming.  The buffer is somewhere between 700-1056 bytes long.

Reading from the buffer: `4e 02 00 00 00 00 {x1 x0}`   -- reads {x1 x0} bytes from the Infectus buffer.  In the official Infectus software, this is always {02 10}, or 528 bytes.  Therefore, to read a 2112 -byte flash page (2048 + 64), you must call this command 4 times.   It also is possible to  use {02 c0} (704 bytes), which will allow all 2112 bytes to be read in 3 chunks.

Writing to the buffer: `4e 01 00 00 00 00 {x1 x0}` -- writes {x1 x0} bytes to the Infectus buffer.  The same rules for block size apply as with reading.