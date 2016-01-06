# Introduction #
This utility is meant to allow any computer that supports libusb to control an Infectus NAND Flash programmer.  It currently only supports 512MB large-block NAND Flash devices.

It operates on files that are (2048 bytes of data + 64 bytes of "spare" data) by 64 pages / block by 4096 blocks -- so, 553648128 bytes in size.  Smaller or larger files may work, but if your file is not a multiple of 2112 bytes, it is probably of the wrong format.

# Usage #

  * `amoxiflash`: With no arguments, usage information will be displayed.
  * `amoxiflash check <filename>`: This will scan the entire file and verify the ECC (parity) data for each page.  It is normal to see some pages listed as "blank" or "unreadable", but any pages listed as "WRONG" indicate corrupt data.
  * `amoxiflash dump <filename>`: The dump command will cause the entire chip to be read and "dumped" to the specified file.  ECC data will also be checked.
  * `amoxiflash program <filename>`: This command will verify the contents of the connected NAND Flash chip against the specified file.  If the contents differ, it will erase that part of the NAND Flash chip and program it using the contents of the file.

This is still experimental software.  If you encounter any problems, please submit a bug report at http://code.google.com/p/amoxiflash/issues.   Patches for bugs or missing features would be greatly appreciated.

-bushing