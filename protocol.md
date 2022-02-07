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

## Table of Contents



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



# Other Resources
* [KVX File Format Specification](https://github.com/piqueserver/aosprotocol/edit/master/index.md) - A mirror of the readme for Slab6 which contains the .kvx file format, the format that the AoS model format is based on
* [VXL File Format Specification](http://silverspaceship.com/aosmap/aos_file_format.html) - A description of the .vxl file format, the format used for AoS maps
