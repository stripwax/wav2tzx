WAV2TZX. Version 0.6
--------------------


LOG
---

This is a minor update to the original voc2tzx utility, version 0.54b. That version was compiled under Watcom C for DOS environment, and it only supported VOC files.

Changes for version 0.6 are listed below:
- Removed VOC support in favor of WAV files (linear PCM encoding, little endian)
- Added option to perform a band-pass filtering to the incoming data (-filter)
- Some code cleanups. Seems that the Watcom compiler assumed every char variable had no sign!

To do:
- Fix endian depedencies. This version only works properly on little endian systems (i.e, PC's, Alpha workstations and Intel Macintosh systems)
- Fix turbo loading algorithms. I have been unable to make a working TZX off of a turbo saved program. Standard speed loaders works well.


INSTALLATION
------------

- Compile wav2tzx.c if there's not a binary for your platform
  $ gcc -o wav2tzx wav2tzx.c

- Copy to your "bin" directory (/usr/bin, /usr/local/bin, etc...) or C:\WINDOWS for Win32 environment.


USAGE
-----

Type wav2tzx without arguments to show the help.

ZXTape Utilities - WAV to TZX Converter v0.6
(c)1997 Tomaz Kac and Martijn van der Heide
(c)2008 Miguel Angel Rodriguez Jodar

Usage: WAV2TZX [switches] INPUT.WAV [OUTPUT.TZX | OUTPUT.TAP]
       Switches:  -tap      Create a .TAP file instead of .TZX
                  -cpc      Create an Amstrad CPC .CDT file
                  -rom      Use ROM timings for 0 and 1 bit
                  -ignore   Ignore Last Byte if it has less than 8 bits
                  -filter   Apply a Butterworth BP filter from 400Hz to 4KHz
                  -noaprox  DON'T make Bit1 h-p == 2*Bit0 h-p
                  -maxp  n  In % - Max. Treshold between Two HPs of Pilot
                  -diff  n  In % - Min. Treshold between 0 and 1 bit HPs
                  -end   n  In % - Min. Treshold between Two HPs to END block
                  -pilot n  In HPs - Min. Number of Pilot HPs for a block
                  -std   n  In % - Max. Treshold for Timing to be Standard
                  -force n  In HPs - Force Length of Custom Pilot Tone to n
                  -bleep    Use the BleepLoad algorythm for conversion
                  -alter    Use Alternative way to recognise 0 or 1 bit!
                  -sync     Force SYNC pulses to Standard values!
                  -middle   Use Middle Value of bit 0 and 1 !
                  -slockX n SpeedLock X (1-4), Variant n (Type 1 and 3 only!)
                  -slskip n Skip n Blocks to get to first Speedlock Turbo block
                  -diload   Use Digital Integration Loader Algorythm


As I am not the original author of this program, there are switches that I don't know how to use. I will describe those that I use daily (in case they are not well explained in the embedded help):

-rom : Force WAV2TZX to not try to guess frequencies for 0 and 1. Instead of that, use standard values. Usefull if we are sure that the audio file contains only standard blocks, because the program won't get fooled if it calculates a different duration for 0 and 1.

-ignore : After the end of a data block, there's always some noise. A real ZX Spectrum ignores that noise because it knows by advance how many bytes it has to load, and stops listeting from the EAR socket when the required number of bytes have been loaded.
WAV2TZX has not that information, so its only way to guess that a block has ended is to check if the last byte it tried to load is complete. If it is not, then, it is surely because that presumed "last byte" was in fact some noise that the program misinterpreted as the beginning of a new byte. It's recommended to use this switch.

-filter : (NEW on 0.6). Performs a 2nd order band-pass Butterworth filter, from 400Hz to 4000Hz. I have added this because there's no way to read input data if it contains glitches, hi freq noise, or noise from mains (50/60Hz). It's recommended to use this switch. WARNING: the WAV file has to be recorded at 44100Hz to use this switch.


EXAMPLES
--------

To make a TAP file off of a WAV file named "prog.wav", which contains audio recorded data from a tape at 44,1kHz, and that data has been recorded using the ROM save routines, do:

wav2tzx -rom -ignore -filter -tap prog.wav


The program will show a table with all found blocks, like this:

ZXTape Utilities - WAV to TZX Converter v0.6
(c)1997 Tomaz Kac and Martijn van der Heide
(c)2008 Miguel Angel Rodriguez Jodar
Loaded WAVE file. Sample rate: 44100 Hz, 1 channels, 16 bits per sample.
Filtering ...
Creating .TAP file ...

 P:demo_intro P-2143,7845 S-1111/ 317 0- 878,1-1755 F-00 B8 L-   19 CY P-1.000
 ------------ P-2143,3140 S-1111/ 317 0- 874,1-1747 F-FF B8 L-   81 CY P-1.000
 B:titulo     P-2143,7850 S-1111/ 317 0- 872,1-1744 F-00 B8 L-   19 CY P-1.000
 ------------ P-2143,3140 S-1111/ 317 0- 864,1-1727 F-FF B8 L-31806 CY P-1.000
 P:loader     P-2143,7850 S-1111/ 317 0- 869,1-1736 F-00 B8 L-   19 CY P-1.000
 ------------ P-2143,3140 S-1111/ 317 0- 873,1-1746 F-FF B8 L-   73 CY P-1.000
 B:C:\COMP\zx P-2143,7850 S-1111/ 317 0- 878,1-1755 F-00 B8 L-   19 CY P-1.000
 ------------ P-2143,3140 S-1111/ 317 0- 865,1-1729 F-FF B8 L-25072 CY 

Each line is divided in several columns:

1111111111111 22222222222 33333333333 4444444444444 5555 66 7777777 88 9999999
 P:loader     P-2143,7850 S-1111/ 317 0- 869,1-1736 F-00 B8 L-   19 CY P-1.000

1: For standard headers, show the type of block (P:program, N:number array, C:character array, B:bytes) and the name (up to 10 characters). For standard blocks, this field contains --------------

2: Information about the pilot tone (P-). Two numbers follow the P letter. The first one is the number of T-states a half wave comprises. The second number is the number of full waves a pilot has. The frequency of the pilot tone can be calculated as 3500000 / (first number * 2). The pilot tone length in seconds can be calculted as (second number * first number ) / 3500000 as well.

3: Information about the sync pulse (S-). Two numbers, following the same scheme as the pilot tone.

4: Information about how to encode 0 and 1. The numbers following 0- and 1- are the number of T-states a half wave comprises for a 0 or 1 respectively.

5: Information about the flag (standard loaders): 00 for header, and different from 00 (normally FF) for data.

6: Sorry, I don't know what this is.

7: Lenght of block, including flag byte and checksum (for standard loaders)

8: CY indicates that a valid checksum has been calculated for this header/block. CN indicates that a checksum has failed. A program loaded with standard ROM routines has to be a valid checksum.

9: Indicates how many silence seconds follow this block.


After the conversion, a file named prog.wav.tap will be created.


CREDITS
-------

(c)1997 Tomaz Kac and Martijn van der Heide
Watcom version (0.54b) of the original VOC2TZX utility. There's no information in the source code about who was the original author of this utility.

(c)2008 Miguel Angel Rodriguez Jodar
Responsible for changes from 0.45b to 0.6 (see top of file)


LICENSE
-------
Unless otherwise stated, this work is put under the GPL license. I don't know the original license of this utility.


CONTACT
-------
Miguel Angel: 	http://www.zxprojects.com
		zxspectrum@atc.us.es

