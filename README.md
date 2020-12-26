# KC Chess

## History

The KC Chess program was written by Kevin Phillips and Craig Bruce 
in Turbo Pascal 5.5 for MS-DOS Computers with 640K of RAM and VGA graphics.

Folder `original` contains original code, help and executables 
(tested under DOSBox). 

Project report - [KC Chess](https://github.com/JulStrat/kcchess/blob/master/KC-Chess_Report.md).

Object Pascal patch by I. Kakoulidis.

<img src="https://github.com/JulStrat/kcchess/blob/master/kcchess.png">
<img src="https://github.com/JulStrat/kcchess/blob/master/board.png">
<img src="https://github.com/JulStrat/kcchess/blob/master/udosbox.png">

## Build

FPC - `fpc -B CHESS.PAS`.
Delphi - `dcc64 -B -NSSystem CHESS.PAS`.

## Requirements

[SDL_bgi](http://libxbgi.sourceforge.net) v2.4.1, [SDL2](https://www.libsdl.org/) Windows dynamic libraries.
