This page documents the networking Protocol of Ace of Spades 0.80

# Overview
[Ace of Spades](http://buildandshoot.com/) uses the [ENet networking
library](http://enet.bespin.org/Features.html) for all server-client
communication. The initial source for the protocol information was the original
[pyspades](http://code.google.com/p/pyspades/) source code, for which the
source for 1.0 was not released. Nonetheless, the 1.0 alpha client has been
reverse engineered to document the protocol.

## Versions

 * [0.8 documentation](protocol080.html)

### Implementers
 Yes or No in is used to specify if server or client wants to or already support 0.8 protocol
 * [OpenSpades](https://github.com/yvt/openspades) (No comment for now from dev)
 * [piqueserver](https://github.com/piqueserver/piqueserver) (No comment for now from dev)
 * [BetterSpades](https://github.com/xtreme8000/BetterSpades) (No comment for now from dev)
 * [SpadesX](https://github.com/SpadesX/SpadesX) (Yes)

Links to the respective projects pages that detail the extensions evailable in
each version should be linked here.

## Other Protocols

 * [Master Server Protocol](protocolmaster.html)
 * [Ping Protocol](protocolping.html)

# Other Resources
* [KVX File Format Specification](https://web.archive.org/web/20100102023608/http://mystaddict.tlayeh.com/Computer%20Camp/Slab6/slab6.txt) - An archive of the mirror of the readme for Slab6 which contains the .kvx file format, the format that the AoS model format is based on
* [VXL File Format Specification](mapformat.html) - A description of the .vxl file format, the format used for AoS maps<br />([original](http://silverspaceship.com/aosmap/aos_file_format.html), [mirror](aos_file_format.html))
