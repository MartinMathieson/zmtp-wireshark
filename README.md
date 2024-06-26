ZMTP Wireshark Dissector
========================

*THIS LUA DISSECTOR WILL BE OBSOLETED!!!!*

*A built-in dissector, written in C, has been merged into master in April 2024, see
https://gitlab.com/wireshark/wireshark/-/merge_requests/14947.  It will mostly likely
become part of an official release towards the end of 2024.*

*This dissector has been renamed to "zmtp-lua" (and the dissector table has been renamed to "zmtp-lua.protocol"),
to allow this dissector to load without clashing names with the C version.
Existing port registrations should continue to work
(where you can register for the table for either version),
you can also still do decode-as for either protocol.  The built-in version additionally has a table in the
preferences that will survive restarts.  Please do report on any regressions seen from this implementation -
feedback was sought and would still be welcomed (https://gitlab.com/wireshark/wireshark/-/issues,
or ask.wireshark.org, or the the mailing lists)*


This is a Lua dissector written for the ZMTP protocol. It supports both the "new" protocol (ZMTP
[version 3.0][zmtp30] and later), as well as the older [version 2][zmtp2].

It supports the [NULL][zmtp30] and [PLAIN][plain] authentication mechanisms.

[zmtp2]: http://rfc.zeromq.org/spec:15
[zmtp30]: http://rfc.zeromq.org/spec:23
[zmtp31]: http://rfc.zeromq.org/spec:37
[plain]:  http://rfc.zeromq.org/spec:24

Screenshot
----------

![Screenshot](/screenshot.png)

Installation
------------

This dissector requires Lua 5.2 or newer.

    mkdir -p ~/.config/wireshark/plugins
    git clone https://github.com/whitequark/zmtp-wireshark ~/.config/wireshark/plugins/zmtp-wireshark

Usage
-----

As ZeroMQ ports are inherently application-specific, you need to use "Decode As -> ZMTP-LUA" on your
zeromq packets. Alternatively, subdissectors can register the ZMTP dissector on specific TCP ports
to automate decoding.

You can use expression `zmtp-lua` to filter packets. TCP segments are automatically reassembled.

If you get frame errors, especially when capturing on `lo`, the problem is that libpcap cannot
capture packets over 64 KiB (relevant [bug](https://github.com/the-tcpdump-group/tcpdump/issues/389));
do `sudo ip link set lo mtu 65500`.

Subdissectors
-------------

This dissector supports calling subdissectors for an application-level protocol. As ZMTP does
not have a generic way of specifying the inner protocol, the mapping is done using TCP ports.

A subdissector that wishes to observe ZMTP frames must register itself in the `zmtp-lua.protocol`
dissector table, using the TCP port as a key. Both source and dest ports are checked, so
bidirectional links (request/response, for example) will need a dissector that can decode both
directions.

    -- Register a subdissector "my_subdissector" to the ZMTP protocol table for TCP port 1234
    local zmtp = DissectorTable.get("zmtp-lua.protocol")
    zmtp:add(1234, my_subdissector_proto)
    -- Register the ZMTP dissector as the default for that TCP port (so no "decode as" is needed)
    local zmtp_dissector = Dissector.get("zmtp-lua")
    local tcp_table = DissectorTable.get("tcp.port")
    tcp_table:add(1234, zmtp_dissector)

License
-------

See [LICENSE](LICENSE.txt).

Acknowledgements
----------------

This dissector is based on a dissector for ZMTP 2, written by [Robert G. Jakabosky](mailto:bobby@neoawareness.com).
