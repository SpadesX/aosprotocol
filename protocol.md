This page documents Ace of Spades 0.8 The newly introduced protocol.

Stearing somewhat away from 0.7x and enforcing some things on
clients and servers.

0.7x protocols were developed in around 2013.
Networks have evolved and our optimizations and performance as well.

0.8 starts from scratch and builds a functional base thats rock solid
and builds the rest of the features from 0.7x on top of this while hoping
to keep most packet IDs and if no changes are needed also content intact.

# Connection

When you connect, you must send a version number as the initial data.

Following that a client needs to send an Existing Player data packet to send
its own name, team etc.

If the client does not send an Existing Player packet first, but any other
packet, then the server closes the connection and seems to temporarily ban the
player.

| Number | AoS version |
|--------|-------------|
| 5      | 0.80        |

Send this magic number as part of the `enet_host_connect(ENetHost, ENetAddress,
channels, int)` function

## Disconnect Reasons

Whenever the connection is closed by the server, there is a reason supplied to
the client in the event's data (event.data).

| Number | Reason                       | Note                                |
|--------|------------------------------|-------------------------------------|
| 1      | Banned                       |                                     |
| 2      | IP connection limit exceded  |                                     |
| 3      | Wrong protocol version       |                                     |
| 4      | Server full                  |                                     |
| 10     | Kicked                       | Check message packet for more info  |
| 11     | Custom                       | Check message packet for more info  |

## About Coordinates

In Ace of Spades the up-down axis is Z and it is inverted. This means 63 is
water level and 0 is the highest point on a map.

# Packets

All packets start with an unsigned byte to specify their type, followed by the
data for that type of packet. The size given for each packet below includes
this byte.

## Data types

Generally, all fields in the Protocol are Low Endian if not specified otherwise.

All data types except float and string can be used for bit packing
and all optimizations. Not limited to numbers and etc.

The following shorthands are used in this document:

| Shorthand    | details                                                 |
| -----------: | ----------                                              |
| Int8         | 8bit signed data type                                   |
| Uint8        | 8bit unsigned data type                                 |
| Int16        | 16bit signed data type                                  |
| Uint16       | 16bit unsigned data type                                |
| Int32        | 32bit signed data type                                  |
| Uint32       | 32bit unsigned data type                                |
| Int64        | 64bit signed data type                                  |
| Uint64       | 64bit unsigned data type                                |
| Float        | 32bit IEEE float                                        |
| CP437 String | String encoded with CP437. Usually fixed-length.        |

## Table of Contents

* [State Update](#state-update)

## Packet Types

This section describes each packet type and the data it contains.

## State Update
`Client <-> Server`

State Update packet is a combination of Input, Position, Orientation and Velocity.
Check handling requirements in:
* [Packet Handling Requirements](#packet-handling-requirements)

| Info       | ID |
|------------|----|
| Packet ID  | 0  |

### State Data

|Field Name|  Field   |                        Notes                        |
|---------:|----------|-----------------------------------------------------|
| Still    |  Uint8   | If set to 1. This is end of the packet              |
| Pos X    |  Float   | Position float for X axis                           |
| Pos Y    |  Float   | Position float for Y axis                           |
| Pos Z    |  Float   | Position float for Z axis                           |
| Ori X    |  Float   | Orientation float for X axis                        |
| Ori Y    |  Float   | Orientation float for Y axis                        |
| Ori Z    |  Float   | Orientation float for Z axis                        |
| Vel X    |  Float   | Velocity float for X axis                           |
| Vel Y    |  Float   | Velocity float for Y axis                           |
| Vel Z    |  Float   | Velocity float for Z axis                           |

### Input States

| Placement | Key    |
| --------- | ------ |
| 1         | up     |
| 2         | down   |
| 3         | left   |
| 4         | right  |
| 5         | jump   |
| 6         | crouch |
| 7         | sneak  |
| 8         | sprint |

### Player State

|Field Name| Field Type  |                        Notes                        |
|---------:|-------------|-----------------------------------------------------|
| ID       |  Uint8      | ID of the player for this data                      |
| Input    |  Uint8      | Bit Packed input. See Input States for details      |
| State    |  State Data | See state data for details                          |


Now for the actual Client and Server packet versions.
They are split as to save bandwidth.

Client version sends sequence which is an 32bit number starting at 0
and increments per 1 each time a new state data packet is sent to server.

Client and Server sequences are SEPARATE

Client version sends an 30 element uint8 array of input states for the last 30
state updates. The first element (0) is the newest and last (29) is the oldest.

Client version then sends state data. If Still flag is set to 1 we do not read
the following 36bytes.
Then client version sends state for the oldest input in the array of inputs.
The still flag in this must ALWAYS be 0.

Server version is similar.

Sequence.
Size of the following array.

More about the size of this array and what determines it in packet handling requirements.

One element contains:
ID of the player
Input state
State Data


### Client Version

Client->Server

| Info       | Size      |
|------------|-----------|
|Total Size: | 107 bytes |

|Field Name|    Field Type     |                        Notes                        |
|---------:|-------------------|-----------------------------------------------------|
| Sequence |  Uint32           | Index of sequence of State data since beginning     |
| Input    |  30 Uint8 Array   | Array of input data. See input states for details   |
| State    |  State Data       | See state data for details                          |
| State-30 |  State Data       | State data for 30th input. Doesnt include still byte|

### Server Version

Server->Clients

| Info       | Size     |
|------------|----------|
|Total Size: | At max 1152 bytes |

|  Field Name   |    Field Type     |                        Notes                        |
|--------------:|-------------------|-----------------------------------------------------|
| Sequence      |  Uint32           | Index of sequence of State data since beginning     |
| Size of array |  Uint8            | Size of the following array.                        |
| Player Data   |  Player State     | See Player State for details                        |

# Packet Handling Requirements

## State Update Requirements

To be filled in later. Its too late, dont wanna screw up.

# Other Resources
* [KVX File Format Specification](https://github.com/piqueserver/aosprotocol/edit/master/index.md) - A mirror of the readme for Slab6 which contains the .kvx file format, the format that the AoS model format is based on
* [VXL File Format Specification](http://silverspaceship.com/aosmap/aos_file_format.html) - A description of the .vxl file format, the format used for AoS maps
