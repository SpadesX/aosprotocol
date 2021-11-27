This page documents Ace of Spades 0.8 The newly introduced protocol based on 0.75.

# Connection

When you connect, you must send a version number as the initial data.

Following that a client needs to send an Existing Player data packet to send
its own name, team etc.

If the client does not send an Existing Player packet first, but any other
packet, then the server closes the connection and seems to temporarily ban the
player.

| Number | AoS version |
|--------|-------------|
| 3      | 0.75        |
| 4      | 0.76        |
| 5      | 0.80        |

Send this magic number as part of the `enet_host_connect(ENetHost, ENetAddress,
channels, int)` function

## Disconnect Reasons

Whenever the connection is closed by the server, there is a reason supplied to
the client in the event's data (event.data).

| Number | Reason                       |
|--------|------------------------------|
| 1      | Banned                       |
| 2      | IP connection limit exceded? |
| 3      | Wrong protocol version       |
| 4      | Server full                  |
| 10     | Kicked                       |

## About Coordinates

In Ace of Spades the up-down axis is Z and it is inverted. This means 63 is
water level and 0 is the highest point on a map.

# Packets

All packets start with an unsigned byte to specify their type, followed by the
data for that type of packet. The size given for each packet below includes
this byte.

## Table of Contents

* [Position Data](#position-data)
* [Orientation Data](#orientation-data)
* [World Update](#world-update)
* [Input Data](#input-data)
* [Weapon Input](#weapon-input)
* [Hit Packet](#hit-packet)
* [Set HP](#set-hp)
* [Grenade Packet](#grenade-packet)
* [Set Tool](#set-tool)
* [Set Colour](#set-colour)
* [Existing Player](#existing-player)
* [Short Player Data](#short-player-data)
* [Move Object](#move-object)
* [Create Player](#create-player)
* [Block Action](#block-action)
* [Block Line](#block-line)
* [CTF State](#ctf-state)
* [TC State](#tc-state)
* [State Data](#state-data)
* [Kill Action](#kill-action)
* [Chat Message](#chat-message)
* [Map Start (0.75)](#map-start-075)
* [Map Chunk](#map-chunk)
* [Player Left](#player-left)
* [Territory Capture](#territory-capture)
* [Progress Bar](#progress-bar)
* [Intel Capture](#intel-capture)
* [Intel Pickup](#intel-pickup)
* [Intel Drop](#intel-drop)
* [Restock](#restock)
* [Fog Colour](#fog-colour)
* [Weapon Reload](#weapon-reload)
* [Change Team](#change-team)
* [Change Weapon](#change-weapon)
* [Map Cached](#map-cached)

## Data types

Generally, all fields in the Protocol are Low Endian if not specified.

The following shorthands are used in this document:

| Shorthand    | details                                                 |
| -----------: | ----------                                              |
| Byte         | 8 bits of arbitrary data. Usually accompanied by a note |
| UByte        | Unisgned 8 bit number                                   |
| LE Float     | 32bit IEEE float                                        |
| LE Uint      | 32bit unsigned integer                                  |
| CP437 String | String encoded with CP437. Usually fixed-length.        |

## Position Data
`Client <-> Server`

This packet is used to set the players position.
Clients should send Position Packet according to the frame rate.
When server sends position packet to client. Client MUST set
this position to its player.

| Info       | Size     |
|------------|----------|
|Packet ID:  |  0       |
|Total Size: | 13 bytes |

|Field Name|Field Type|Example|Notes|
|---------:|----------|-------|-----|
|        x | LE Float |  `0`  |     |
|        y | LE Float |  `0`  |     |
|        z | LE Float |  `0`  |     |

## Orientation Data
This packet is used to set the players orientation.
Clients should send Orientation packet according to frame rate.
When server sends a client orientation packet client MUST
apply this orientation to its player.

| Info       | Size     |
|------------|----------|
|Packet ID   |  1       |
|Total Size: | 13 bytes |

#### Fields

|Field Name|Field Type|Example|Notes|
|---------:|----------|-------|-----|
|        x | LE Float |  `0`  |     |
|        y | LE Float |  `0`  |     |
|        z | LE Float |  `0`  |     |

## World Update
Updates position and orientation of players.
When a player is no longer visible to other player
both of them should get at least 3 (to account for lost packets)
World Update packets that contain the player out of range
and set orientation and position to {0,0,0} vectors or
other impossible value.

Timestamp should be used to check for order of the packets
as to not set variables that were before already applied values.
This should eliminate jitter in position.


| Info        | Size                                       |
| ----------- | ------------------------------------------ |
| Packet ID   | 2                                          |
| Total Size: | 10 + (Number of player IDs sent * 25) bytes |

#### Fields

| Field Name                         | Field Type                        | Example | Notes                    |
|------------------------------------|-----------------------------------|--------------|--------------------------|
| Number of Player IDs sent          | UByte                             | `32`         |                          |
| Timestamp                          | UInt64                            | `1638026801` | Unix type of epoch       |
| players positions and orientations | Array of Player Position Data     |              | See below table for data |

#### 'Player Position Data'

| Info        | Size      |
| ----------- | --------- |
| Total Size: | 25 bytes  |

#### Fields

| Field Name    | Field Type | Example | Notes   |
|---------------|------------|---------|---------|
| Player ID     | UByte      | `15`    |         |
| x position    | LE Float   | `0`     |         |
| y position    | LE Float   | `0`     |         |
| z position    | LE Float   | `0`     |         |
| x orientation | LE Float   | `0`     |         |
| y orientation | LE Float   | `0`     |         |
| z orientation | LE Float   | `0`     |         |

## Input Data
Contains the key-states of a player, packed into a byte.

| Info        | Size     |
| ----------- | -------- |
| Packet ID   | 3        |
| Total Size: | 3 bytes  |

#### Fields

| Field Name | Field Type | Example | Notes                                                                 |
|------------|------------|---------|-----------------------------------------------------------------------|
| player ID  | UByte      | `0`     |                                                                       |
| key states | UByte      | `0`     | Each bit in the byte represents a key, as defined in the table below. |

#### Key States:

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

## Weapon Input
Contains the weapon input state(?).

| Info        | Size    |
|-------------|---------|
| Packet ID   | 4       |
| Total Size: | 3 bytes |

| Field Name   | Field Type | Example | Notes                                                                                        |
|--------------|------------|---------|----------------------------------------------------------------------------------------------|
| player ID    | UByte      | `0`     |                                                                                              |
| weapon input | UByte      | `0`     | The lowest bit represents the primary fire, the second lowest represents the secondary fire. |

## Hit Packet
#### Client-to-Server

Sent by the client when a hit is registered. The server should verify that this
is possible to prevent abuse (such as hitting without shooting, facing the
wrong way, etc).

| Info        | Size    |
| ----------- | ------- |
| Packet ID   | 5       |
| Total Size: | 3 bytes |

#### Fields

| Field Name    | Field Type | Example | Notes                     |
|---------------|------------|---------|---------------------------|
| player ID hit | UByte      | `0`     |                           |
| hit type      | UByte      | `0`     | See values in table below |

#### Hit Types

| Value | Type  |
| ----- | ----- |
| 0     | torso |
| 1     | head  |
| 2     | arms  |
| 3     | legs  |
| 4     | melee |

## Set HP
#### Server-to-Client

Sent to the client when hurt.

| Info        | Size     |
| ----------- | -------- |
| Packet ID   | 5        |
| Total Size: | 15 bytes |

| Field Name        | Field Type | Example | Notes                |
|-------------------|------------|---------|----------------------|
| HP                | UByte      | `0`     |                      |
| type              | UByte      | `0`     | 0 = fall, 1 = weapon |
| source x position | LE Float   | `0`     |                      |
| source y position | LE Float   | `0`     |                      |
| source z position | LE Float   | `0`     |                      |

## Grenade Packet
Spawns a grenade with the given information.

| Info         | Size      |
| ------------ | --------- |
| Packet ID    | 6         |
| Total Size:  | 30 bytes  |

#### Fields

| Field Name  | Field Type | Example | Notes |
|-------------|------------|---------|-------|
| player ID   | UByte      | `0`     |       |
| fuse length | LE Float   | `0`     |       |
| x position  | LE Float   | `0`     |       |
| y position  | LE Float   | `0`     |       |
| z position  | LE Float   | `0`     |       |
| x velocity  | LE Float   | `0`     |       |
| y velocity  | LE Float   | `0`     |       |
| z velocity  | LE Float   | `0`     |       |

## Set Tool
Sets a player's currently equipped tool/weapon.

| Info        | Size    |
|------------ |---------|
| Packet ID   | 7       |
| Total Size: | 3 bytes |

#### Fields

| Field Name | Field Type | Example | Notes                        |
|------------|------------|---------|------------------------------|
| player ID  | UByte      | `0`     |                              |
| tool       | UByte      | `0`     | Tool values are listed below |

#### Tools

| Value | Type    |
| ----- | -----   |
| 0     | spade   |
| 1     | block   |
| 2     | gun     |
| 3     | grenade |

## Set Colour
Set the colour of a player's held block.

| Info        | Size    |
|------------ |---------|
| Packet ID   | 8       |
| Total Size: | 5 bytes |

| Field Name | Field Type | Example | Notes |
|------------|------------|---------|-------|
| player ID  | UByte      | `0`     |       |
| blue       | UByte      | `0`     |       |
| green      | UByte      | `0`     |       |
| red        | UByte      | `0`     |       |

## Existing Player
Set player's team, weapon, etc.

| Info        | Size    |
|------------ |---------|
| Packet ID   | 9       |
| Total Size: | depends |

#### Fields

| Field Name | Field Type                                                 | Example | Notes |
|------------|------------------------------------------------------------|---------|-------|
| player ID  | UByte                                                      | `0`     |       |
| team       | Byte                                                       | `0`     |       |
| weapon     | UByte                                                      | `0`     |       |
| held item  | UByte                                                      | `0`     |       |
| kills      | LE UInt                                                    | `0`     |       |
| blue       | UByte                                                      | `0`     |       |
| green      | UByte                                                      | `0`     |       |
| red        | UByte                                                      | `0`     |       |
| name       | [CP437](http://en.wikipedia.org/wiki/Code_page_437) String | `Deuce` |       |

## Short Player Data
Like Existing Player, but with less information.

| Info        | Size    |
|------------ |---------|
| Packet ID   | 10      |
| Total Size: | 4 bytes |

#### Fields

| Field Name | Field Type | Example | Notes |
|------------|------------|---------|-------|
| player ID  | UByte      | `0`     |       |
| team       | Byte       | `0`     |       |
| weapon     | UByte      | `0`     |       |

## Move Object
This packet is used to move various game objects like tents, intels and even grenades. When moving grenades in TC mode the voxlap client has a bug that changes grenades' models to small tents.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 11       |
| Total Size: | 15 bytes |

#### Fields

| Field Name | Field Type | Example | Notes       |
|------------|------------|---------|-------------|
| object id  | UByte      | `0`     |             |
| team       | UByte      | `0`     | 2 = neutral |
| x position | LE Float   | `0`     |             |
| y position | LE Float   | `0`     |             |
| z position | LE Float   | `0`     |             |

## Create Player
Send on respawn of a player.

| Info        | Size    |
| ----------  | ------- |
| Packet ID   | 12      |
| Total Size: | depends |

#### Fields

| Field Name | Field Type                                                 | Example | Notes |
|------------|------------------------------------------------------------|---------|-------|
| player id  | UByte                                                      | `0`     |       |
| weapon     | UByte                                                      | `0`     |       |
| team       | Byte                                                       | `0`     |       |
| x position | LE Float                                                   | `0`     |       |
| y position | LE Float                                                   | `0`     |       |
| z position | LE Float                                                   | `0`     |       |
| name       | [CP437](http://en.wikipedia.org/wiki/Code_page_437) String | `Deuce` |       |

## Block Action
Sent when a block is placed/destroyed.

| Info        | Size        |
| ----------- | ----------- |
| Packet ID   | 13          |
| Total Size: | 16/19 bytes |

#### Fields

| Field Name  | Field Type | Example | Notes           |
|-------------|------------|---------|-----------------|
| player id   | UByte      | `0`     |                 |
| action type | UByte      | `0`     | See table below |
| silent      | UByte      | `0`     |                 |
| x position  | LE Int     | `0`     |                 |
| y position  | LE Int     | `0`     |                 |
| z position  | LE Int     | `0`     |                 |
| Blue color  | UByte      | `0`     | sent for type 4 |
| Green color | UByte      | `0`     | sent for type 4 |
| Red color   | UByte      | `0`     | sent for type 4 |


#### Fields

| Value | Type                                  | Notes                                               |
| ----- | ------------------------------------- | --------------------------------------------------- |
| 0     | build                                 | places a block with the player's selected color     |
| 1     | bullet and spade(left button) destroy |                                                     |
| 2     | spade(right button) destroy           | destroys 3 blocks, one above and below additionally |
| 3     | grenade destroy                       | destroys all blocks within an 3x3x3 area            |
| 4     | block change of color                 | Changes color of the block to the color provided    |

## Block Line
Create a line of blocks between 2 points. The block color is defined by the `Set Color` packet. 

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 14       |
| Total Size: | 26 bytes |

| Field Name       | Field Type      | Example | Notes |
| ---------------- | --------------- | ------- | ----- |
| player id        | UByte           | `0`     |       |
| start x position | LE Int          | `0`     |       |
| start y position | LE Int          | `0`     |       |
| start z position | LE Int          | `0`     |       |
| end x position   | LE Int          | `0`     |       |
| end y position   | LE Int          | `0`     |       |
| end z position   | LE Int          | `0`     |       |

## CTF State
Brief description.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | none     |
| Total Size: | 52 bytes |

#### Fields

| Field Name             | Field Type    | Example | Notes                                                                |
| ---------------------- | ------------- | ------- | -------------------------------------------------------------------- |
| team 1 score           | UByte         | `0`     |                                                                      |
| team 2 score           | UByte         | `0`     |                                                                      |
| capture limit          | UByte         | `0`     |                                                                      |
| intel flags            | UByte         | `0`     | bits signal if teams have intel - bit 1 for team 1, bit 2 for team 2 |
| team 1 intel location  | Location Data | `0`     | see below                                                            |
| team 2 intel location  | Location Data | `0`     | see below                                                            |
| team 1 base x position | LE Float      | `0`     |                                                                      |
| team 1 base y position | LE Float      | `0`     |                                                                      |
| team 1 base z position | LE Float      | `0`     |                                                                      |
| team 2 base x position | LE Float      | `0`     |                                                                      |
| team 2 base y position | LE Float      | `0`     |                                                                      |
| team 2 base z position | LE Float      | `0`     |                                                                      |

The intel location data is 12 bytes long. If the intel is being held, the first
byte is a UByte with the id of the holding player, then the rest are padding.
If the intel is on the ground (not being held), the data will hold three LE
Floats with its x, y and z position.

#### Fields

| Intel State         | Field Name        | Field Type   |
| ------------------- | ------------      | ------------ |
| Held                | holding player id | UByte        |
|                     | padding           | 11 bytes     |
| Dropped             | intel x position  | LE Float     |
|                     | intel y position  | LE Float     |
|                     | intel z position  | LE Float     |

This packet is not a complete packet, as it is only sent after the initial
data, where the gamemode is sent. It could be considered as part of that
initial data packet, but as what's sent varies greatly depending on the
gamemode, it is documented separately.

## TC State

| Field Name                | Field Type                          | Example | Notes                                                                      |
| ------------------------- | ----------------------------------- | ------- | -------------------------------------------------------------------------- |
| territory count           | UByte                               | 16      | Maximum is 16 otherwise the client will crash with 'Invalid memory access' |
| Array[] of territory data | LE Float, LE Float, LE Float, UByte |         | See table below                                                            |

This packet is not a complete packet, as it is only sent after the initial
data, where the gamemode is sent. It could be considered as part of that
initial data packet, but as what's sent varies greatly depending on the
gamemode, it is documented separately.

## State Data
`Server-->Client`

Indicates that the map transfer is complete. Also informs the client of
numerous game parameters. Be aware that CTFState or TCState may be appended to
the packet after the gamemode id portion.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 15       |
| Total Size: | 52 bytes |

#### Fields

| Field Name             | Field Type     | Example   | Notes                     |
| ---------------------- | -------------- | --------- | ------------------------- |
| player id              | UByte          | 0         |                           |
| fog (blue)             | UByte          | 0         |                           |
| fog (green)            | UByte          | 0         |                           |
| fog (red)              | UByte          | 0         |                           |
| team 1 color (blue)    | UByte          | 0         |                           |
| team 1 color (green)   | UByte          | 0         |                           |
| team 1 color (red)     | UByte          | 0         |                           |
| team 2 color (blue)    | UByte          | 0         |                           |
| team 2 color (green)   | UByte          | 0         |                           |
| team 2 color (red)     | UByte          | 0         |                           |
| team name 1            | CP437 String   | Blue      | Always 10 characters long |
| team name 2            | CP437 String   | Green     | Always 10 characters long |
| gamemode id            | UByte          | 0         | 0 for CTF, 1 for TC       |


## Kill Action
#### Server->Client

Notify the client of a player's death.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 16       |
| Total Size: | 5 bytes  |

| Field Name       | Field Type | Example | Notes                 |
|------------------|------------|---------|-----------------------|
| player ID        | UByte      | 12      | Player that died      |
| killer ID        | UByte      | 8       |                       |
| kill type        | UByte      | 0       | See table below       |
| respawn time     | UByte      | 1       | Seconds until respawn |

#### Fields

| Value | Type                 |
|-------|----------------------|
| 0     | WEAPON (body, limbs) |
| 1     | HEADSHOT             |
| 2     | MELEE (spade)        |
| 3     | GRENADE              |
| 4     | FALL                 |
| 5     | TEAM\_CHANGE         |
| 6     | CLASS\_CHANGE        |

## Chat Message
#### Two-way

Reasonable limits must placed on length and frequency of chat messages.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 17       |
| Total Size: | . bytes  |

| Field Name   | Field Type                                                 | Example           | Notes           |
|--------------|------------------------------------------------------------|-------------------|-----------------|
| player id    | UByte                                                      | `0`               |                 |
| Chat Type    | UByte                                                      | `0`               | See table below |
| Chat Message | [CP437](http://en.wikipedia.org/wiki/Code_page_437) String | `"join /squad 1"` |                 |



#### Fields

| Value | Type         | voxlap default color            |
| ----- | ------------ | ------------------------------- |
| 0     | CHAT\_ALL    | white                           |
| 1     | CHAT\_TEAM   | team color, black for spectator |
| 2     | CHAT\_SYSTEM | red                             |

## Map Start (0.75)
#### Server->Client

Sent when a client connects, or a map is advanced for already existing connections.

Should be the first packet received when a client connects.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 18       |
| Total Size: | 5 bytes  |

| Field Name | Field Type | Example | Notes |
|------------|------------|---------|-------|
| Map size   | Uint32     | `4567`  |       |

## Map Chunk
#### Server->Client

Sent just after **Map Start**, repeatedly until the entire map is sent.

Should always be the next sequence of packets after a **Map Start** packet.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 19       |
| Total Size: | depends  |

#### Fields

| Field Name | Field Type | Example | Notes                                                                                                                              |
|------------|------------|---------|------------------------------------------------------------------------------------------------------------------------------------|
| Map Data   | UByte      | `0`     | [DEFLATE/zlib](http://en.wikipedia.org/wiki/DEFLATE) encoded [AOS map data](http://silverspaceship.com/aosmap/aos_file_format.html)|

## Player Left
#### Server->Protocol

Sent when a player disconnects.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 20       |
| Total Size: | 2 bytes  |

#### Fields

| Field Name | Field Type | Example | Notes |
|------------|------------|---------|-------|
| player ID  | UByte      | `0`     |       |

## Territory Capture
#### Server->Protocol

Sent when a player captures a Command Post in Territory Control mode.

Captures have affects on the client.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 21       |
| Total Size: | 5 bytes  |

#### Fields

| Field Name | Field Type | Example | Notes                           |
|------------|------------|---------|---------------------------------|
| player ID  | UByte      | `0`     |                                 |
| entity ID  | UByte      | `0`     | The ID of the CP being captured |
| winning    | UByte      | `0`     | (or losing)                     |
| state      | UByte      | `0`     | team id                         |

## Progress Bar
#### Server->Client

Display the TC progress bar.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 22       |
| Total Size: | 8 bytes  |

#### Fields

| Field Name        | Field Type | Example | Notes                                                                                            |
|-------------------|------------|---------|--------------------------------------------------------------------------------------------------|
| entity ID         | UByte      | `0`     | The ID of the tent entity                                                                     |
| capturing team ID | UByte      | `1`     |                                                                                                  |
| rate              | Byte       | `2`     | Used by the client for interpolation, one per team member capturing (minus enemy team members). One rate unit is 5% of progress per second. |
| progress          | LE Float   | `0.5`   | In range [0,1]                                                                                   |

## Intel Capture
#### Server->Protocol

Sent when a player captures the intel, which is determined by the server.

Winning captures have affects on the client.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 23       |
| Total Size: | 3 bytes  |

#### Fields

| Field Name | Field Type | Example | Notes                   |
|------------|------------|---------|-------------------------|
| player ID  | UByte      | `0`     |                         |
| winning    | UByte      | `0`     | Was the winning capture |

## Intel Pickup
#### Server->Protocol

Sent when a player collects the intel, which is determined by the server.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 24       |
| Total Size: | 2 bytes  |

#### Fields

| Field Name | Field Type | Example | Notes |
|------------|------------|---------|-------|
| player ID  | UByte      | `0`     |       |

## Intel Drop
#### Server->Protocol

Sent when a player dropped the intel. This will update the intel position on the client.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 25       |
| Total Size: | 14 bytes |

#### Fields

| Field Name | Field Type | Example | Notes                              |
| player ID  | UByte      | `0`     | ID of the player who dropped intel |
| x position | LE Float   | `32.0`  |                                    |
| y position | LE Float   | `32.0`  |                                    |
| z position | LE Float   | `32.0`  |                                    |

## Restock
#### Server->Protocol

Id of the player who has been restocked.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 26       |
| Total Size: | 2 bytes  |

#### Fields

| Field Name | Field Type | Example | Notes                          |
|------------|------------|---------|--------------------------------|
| player ID  | UByte      | `0`     | ID of the player who restocked |

## Fog Colour
#### Server->Client

Set the colour of a player's fog.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 27       |
| Total Size: | 5 bytes  |

#### Fields

| Field Name | Field Type | Example      | Notes        |
| ---------- | ---------- | ------------ | ------------ |
| fog color  | UInt       | `0h00fefefe` | BGRA encoded |

## Weapon Reload
#### Client-->Server->Protocol

Sent by the client when the player reloads their weapon, and relayed to other
clients after protocol logic applied.

This has no affect on animation, but is used to trigger sound effects on the
other clients.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 28       |
| Total Size: | 4 bytes  |

#### Fields

| Field Name   | Field Type | Example | Notes               |
|--------------|------------|---------|---------------------|
| player ID    | UByte      | `0`     | Player who reloaded |
| clip ammo    | UByte      | `0`     |                     |
| reserve ammo | UByte      | `0`     |                     |

## Change Team
#### Client-->Server-->Protocol-->Kill Action & Create Player

Sent by the client when the player changes team. Is not relayed to all clients
directly, but instead uses **Kill Action**
then **Create Player** to inform other
clients of the team change.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 29       |
| Total Size: | 3 bytes  |

#### Fields

| Field Name | Field Type | Example | Notes                     |
|------------|------------|---------|---------------------------|
| player ID  | UByte      | `0`     | Player who changed team   |
| Team ID    | Byte       | `0`     | See values in table below |

#### Team IDs

| Value | Type      |
|------:|-----------|
| -1    | spectator |
| 0     | blue      |
| 1     | green     |

## Change Weapon
#### Client-->Server-->Protocol-->Kill Action & Change Weapon

Sent by the client when player changes weapon, and relayed to clients by server
after `filter_visibility` logic is applied.

Receiving clients will also be sent a preceding
**Kill Action** to inform them the player
has died both of which are sent as reliable packets.

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 30       |
| Total Size: | 3 bytes  |

| Field Name | Field Type | Example | Notes                       |
|------------|------------|---------|-----------------------------|
| player ID  | UByte      | `0`     | Player who's changed weapon |
| Weapon ID  | UByte      | `0`     | See values in table below   |

#### Weapon ID

| Value | Type    |
|-------|---------|
| 0     | rifle   |
| 1     | smg     |
| 2     | shotgun |

## Map Cached
`Client->Server`

| Info        | Size     |
| ----------  | -------- |
| Packet ID   | 31       |
| Total Size: | 2 bytes  |

| Field Name | Field Type | Example | Notes                        |
|------------|------------|---------|------------------------------|
| Cached     | UByte      | `1`     | `1` if cached, `0` otherwise |

# Other Resources
* [KVX File Format Specification](https://github.com/piqueserver/aosprotocol/edit/master/index.md) - A mirror of the readme for Slab6 which contains the .kvx file format, the format that the AoS model format is based on
* [VXL File Format Specification](http://silverspaceship.com/aosmap/aos_file_format.html) - A description of the .vxl file format, the format used for AoS maps
