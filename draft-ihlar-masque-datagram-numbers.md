---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "A Sequence Number Extension for HTTP Datagrams"
abbrev: "TODO - Abbreviation"
category: info

docname: draft-ihlar-masque-datagram-numbers-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: TSV
workgroup: Masque Working Group
keyword:
 - masque
 - datagram
 - multipath

venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
    fullname: Marcus Ihlar
    organization: Ericsson AB
    email: marcus.ihlar@ericsson.com

normative:

informative:


--- abstract

This document defines a sequence number extension to HTTP datagrams used to carry proxied UDP payload or IP datagrams.
This extension is useful when HTTP datagrams are transported on top of a multipath protocol that does not ensure
in-order delivery as it allows a masque endpoint to implement reordering logic specific to its needs.

--- middle

# Introduction
<!--
When HTTP datagrams are used with the multipath extension to QUIC it is possible that they are sent over two or
more paths simultaneously. Simultaneous transfer over multiple paths can lead to datagrams arriving out-of-order. HTTP
datagrams sent over a QUIC connection can either be encoded as capsules that are sent reliably in QUIC streams or as
QUIC datagrams that do not provide any reliability. Both these "modes" can be problematic when datagrams are used to
proxy IP or UDP. QUIC streams ensure that proxied datagrams are delivered in-order at the cost of introducing buffering
with corresponding delay variation. With QUIC datagrams there will be no delay variations in the proxy, but instead
out-of-order data needs to be handled by the endpoints. -->

This document defines a sequence number extension to HTTP datagrams. Sequence numbers at the HTTP datagram layer allows
a receiving endpoint to implement arbitrary reordering logic, which can be useful when proxied datagrams are sent over
multiple paths simultaneously, e.g. using the multipath QUIC extension. The extension applies to HTTP datagarms when
they are used with the extended CONNECT method and the protocols are either connect-ip or connect-udp.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Sequence Number Datagram Extension

The Sequence Number datagram extension prepends sequence numbers to HTTP datagrams. Datagram sequence numbers are
unsigned integers initiated to 0 and are incremented by 1 for every transmitted HTTP datagram, except for when the
integer overflows and is reset to 0. The extension can be used with the HTTP CONNECT method when the :protocol pseudo
header is equal to "connect-udp" or "connect-ip".

## Registration

Endpoints indicate support for Sequence Number Datagram type by including the boolean-valued Item Structured Field
"DG-Sequence: ?1" in the HTTP Request and Response headers.

A new datagram sequence is registered by sending a REGISTER_SEQUENCE_CONTEXT capsule.

~~~
REGISTER_SEQUENCE_CONTEXT Capsule {
  Type (i) = REGISTER_SEQUENCE_CONTEXT,
  Length (i),
  Context ID (i),
  Payload Context ID (i),
  Representation (8)
}
~~~

The capsule has the following fields:

Context ID: Identifies a sequence number context.

Payload Context ID: Identifies the type of payload that follows a sequence number. The value MUST be equal to a
previously registered Context ID.

Representation: The size in bits of the unsigned interger used to encode the sequence number, the value MUST be one of
the following: 8, 16, 32 or 64.

It is possible to use multiple contexts for a single sequence, this is needed if multiple datagram payload formats are
used in parallel. To add a context to an existing sequence an endpoint sends an ADD_SEQUENCE_CONTEXT Capsule. The
capsule has the following format:

~~~
ADD_SEQUENCE_CONTEXT Capsule {
  Type (i) = ADD_SEQUENCE_CONTEXT,
  Length (i),
  Sequence Context ID (i),
  Context ID (i),
  Payload Context ID (i)
}
~~~

The capsule has the following fields:

Sequence Context: Identifies the original sequence number context.

Context ID: A new context identifier that refers to the same sequence identified by the Sequence Context ID.

Payload Context ID: Identifies the type of payload that follows a sequence number. The value MUST be equal to a
previously registered Context ID.

## Datagram Format

A Sequence Number Datagrams has the following format:

~~~
Sequence Number Datagram {
  Context ID (i),
  Sequence Number (8..64),
  Payload (..)
}
~~~

Context ID: This value indicates that the datagram contains a sequence number and the format of the data that follows
the sequence number.

Sequence Number: Unsigned integer of size specified in registration, indicates the transmission order of the datagagram.

Payload: Datagram payload.

# Security Considerations

Users of the sequence number extension typically maintain a reordering buffer, a malicious endpoint might assign
sequence numbers out-of-order so that the receiver attempts to buffer as many datagrams as possible.


# IANA Considerations

HTTP Header

Capsule

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
