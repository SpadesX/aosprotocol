# VMF (Voxel Map File) format specification 
Very WIP. Until optimized heavily 0.8 will use VXL

#### Revision History

* 13th December 2021: Initial format introduction (DarkNeutrino)

## Introduction

This document describes the format of VMF file used by 0.8 AoS clients and servers

## Data types

Generally, all fields in the VMF are Low Endian if not specified.

The following shorthands are used in this document:

| Shorthand    | details                                                 |
| -----------: | ----------                                              |
| Byte         | 8 bits of arbitrary data. Usually accompanied by a note |
| UByte        | Unisgned 8 bit number                                   |
| LE Float     | 32bit IEEE float                                        |
| LE Ushort    | 16bit unsigned number                                   |
| LE Uint      | 32bit unsigned integer                                  |
| LE Uint64    | 64bit unsigned integer                                  |
| ASCII String | ASCII string. Must include ending character \0.         |

## Format

The VMF format is quite simple. It contains the metadata and basic map information
and the rest are blocks.

### Header

At start of the file we have the header

|     Field Name       |Field Type |Example           |      Notes     |
|---------------------:|-----------|------------------|----------------|
| Map Name             | String    |  `MapName`       | Char limit: 32 |
| Author               | String    |  `DarkNeutrino`  | Char limit: 32 |
| Recommended GameMode | UByte     |  `0`             | Check GM list  |
| X Size               | LE Uint   |  `1024`          |                |
| Y Size               | LE Uint   |  `2048`          |                |
| Z Size               | LE Ushort |  `512`           |                |
| Team1 Spawn X Start  | LE Uint   |  `256`           |                |
| Team1 Spawn X End    | LE Uint   |  `512`           |                |
| Team1 Spawn Y Start  | LE Uint   |  `256`           |                |
| Team1 Spawn Y End    | LE Uint   |  `512`           |                |

So the entire size of header is 728bits.
Should be noted that server and client are responsible for checking where
either map name or author strings end.
If ending character '\0' is not found in the 32 characters for either string
then server must switch to other map or stop execution.
Client should in this same case disconnect from the server and notify
user of this event.

### GameModes list

| GameModes | ID |
| --------- | -- |
|    CTF    | 0  |
|    TC     | 1  |
|   Babel   | 2  |
|   Arena   | 3  |

This is a basic gamemode list. If you would like to add another please
submit a Pull Request.

### Map Voxels

Each voxel is 64bit. Even if not yet utilized fully it should be
64bits as to make sure we have a reserve for future. Unfilled
bits should be set to 0.

|   Field Name   | Field Type |              Notes               |
|----------------|------------|----------------------------------|
|      Red       | UByte      | Red color channel                |
|     Green      | UByte      | Green color channel              |
|     Blue       | UByte      | Blue color channel               |
|     Light      | UByte      | Amount of light emitted          |
|   Voxel Type   | UByte      | Check voxel type list            |
| Indestructible | UByte      | 1 if indestructible. 0 otherwise |
|    RESERVED    | LE Ushort  | 16bits reserved for future usage |

I think the Red, Green, Blue are just 0-255 values representing color combined.

The Light is a value representing how much light should be emitted.
This is not alpha channel. Neither should it be used for coloring
touching voxels. Clients should implement lighting from blocks.

Voxel type is to tell client or server what type of voxel this voxel is.

|  Voxel Type  | ID |
| ------------ |----|
|      Air     | 0  |
|     Solid    | 1  |
|     Water    | 2  |

Client is responsible for checking which voxel is visible as to not
render unneeded voxels.
There is also a water voxel. As to avoid water fluid simulation each
adjacent voxel is indestructible. Except for top ones. Which can be
destroyed as to reveal the water unless the indestructible bit is set
for this voxel.

Indestructible as the name suggests is used to say if voxel should be
destructible by player or other way.

The rest of the 64 bits is reserved for future usage.
The bits in the reserve should be set to 0.

### Coordinate system and limits

The coordinate system is XYZ.
Where 000 is bottom left front corner.

//Add coordinate system picture

And since player is 3 voxels in height the highest point in map should be 4 voxels below the actual max Z limit.
So if we say 0 is the lowest point in map and 63 is the highest point then the highest solid voxel should be
at Z 59. The player feet at Z60. Torso at Z61 and head at Z62. That way even when the player jumps while standing
on the highest solid block the player head WONT go outside of the map limits.

Clients should also NOT allow placing blocks above this limit. Same goes for server.

### Layout

The way the map is structured is as memory.

As for the map array it is an array where each index is of size 64bits.

The size of the array is taken from size of XYZ multiplied together and all multiplied by 64.
So for example if the X is 512, Y is 512 and Z is 64 then the size of the array would be
(512 * 512 * 64) * 64 which is 1073741824bits or if represented in MB its 134.217728MB.

The fields are all stored in these 64bits and also ordered as specified above.
Client or server is responsible for splitting these 64bits into their respective variables.
The same applies to header as well.

The map array in C would be defined as such:

uint64 map[X][Y][Z];

The header is added on top of the map array memory and sent as such to clients or written to a file.

## Example Code

To be added later

## Credits
* [Ace of Spades](http://ace-spades.com) is by Ben Aksoy.

* VMF (Voxel Map File) by DarkNeutrino.
