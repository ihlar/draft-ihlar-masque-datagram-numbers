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
category: std

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
  mail: masque@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/masque/
  github: ihlar/draft-ihlar-masque-datagram-numbers
  # latest: https://example.com/LATEST

author:
-
    fullname: Marcus Ihlar
    organization: Ericsson AB
    email: marcus.ihlar@ericsson.com
-
   ins:  M. Westerlund
   name: Magnus Westerlund
   org: Ericsson
   email: magnus.westerlund@ericsson.com

normative:

informative:
  3GPP TS 23.501:
    target: https://www.3gpp.org/ftp/Specs/archive/23_series/23.501/23501-i22.zip
    title: System architecture for the 5G System (5GS) - Release 18
    # seriesinfo:
    #   ANSSI Technical Report DAT-NT-003
    author:
      -
        ins: 3rd Generation Partnership Project
    date: July 2023

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

This document defines a sequence number extension to HTTP datagrams {{!RFC9297}}. Sequence numbers at the HTTP
datagram layer allows a receiving endpoint to implement arbitrary reordering logic, which can be useful when proxied
datagrams are sent over multiple paths simultaneously, such as when using the multipath QUIC extension
{{?MPQUIC=I-D.ietf-quic-multipath-04}}. The extension applies to HTTP datagrams when they are used with the extended
CONNECT method and the protocols are either connect-ip {{!CONNECT-IP=I-D.ietf-masque-connect-ip}} or connect-udp
{{!RFC9298}}.

## ATSSS

The motivation for this extension comes from the Access Traffic Steering, Switching and Splitting (ATSSS) feature
defined for the 5G System by 3GPP in section 5.32 of {{?3GPP TS 23.501}}.

ATSSS is an optional feature in the 5G system that enables concurrent use of 3GPP and non-3GPP accesses with a single
PDU session. A set of steering functionalities and steering modes that determine the types of concurrent path usage
supported by ATSSS. As of Release 18 of the 5G System Architecture specification there are three steering
functionalities defined for ATSSS: ATSSS-LL, MPTCP and MPQUIC. ATSSS-LL is a "Lower Layer Functionality" that operates
below the IP layer, it can be used to steer, switch and split all types of traffic including both IP and Ethernet PDU
Sessions. MPTCP and MPQUIC are so called "Higher Layer Functionalities" and operate above the IP layer to steer, switch
and split TCP and UPD traffic respectively.

The MPQUIC steering functionality makese use of multipath capable HTT3 proxies that support the extended CONNECT method
with the connect-udp protocol. The MPQUIC steering mode defines two datagram modes that are used for encapsulation of
UDP payload. The default mode is to send HTTP datagrams unreliably over QUIC datagrams. The second, optional mode is to
encapsulate UDP payload in HTTP datagrams that are extended with sequence numbers. The mode to use is decided based on
the steering mode in use and application requirements through the use of ATSS Rules.

The different steering modes determine the way traffic flows make use of concurrent paths. There are two modes where
the use of sequence numbers is beneficial: Load Balancing traffic steering and Redundant traffic steering.

The Load Balancing steering mode implies parallel transmission over the 3GPP and non-3GPP accesses, this mode is
commonly known as bandwidth aggregation. Splitting a data transmission over multiple pahts while increasing the
available bandwith, often leads to packets being delivered out-of-order. Whether the packet disorder is a much of a
problem depends on the properties of the protocols and applications carried over the proxied payload. Large degrees of
reordering might trigger spurious packet loss detection. By buffering data received out-of-order an ATSSS endpoint can
minimize the amount of data delivered out-of-order to the final endpoints.

Redundant traffic steering implies duplication of traffic over the 3GPP and non-3GPP accesses. Such a steering mode,
while costly, is useful for applications and users with strong requirements on availability.  When data is duplicated
at one end it needs to be de-duplicated at the other end. There are many ways de-duplication can be achieved, sequence
numbering is one of them.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Sequence Number Datagram Extension

The Sequence Number datagram extension prepends sequence numbers to HTTP datagrams. Datagram sequence numbers are
unsigned integers initiated to 0 and are incremented by 1 for every transmitted HTTP datagram, except for when the
integer overflows and is reset to 0. The extension can be used with the HTTP CONNECT method when the :protocol pseudo
header is equal to "connect-udp" or "connect-ip". Use of the sequence number extension is determined per request, and
the scope of a datagram sequence is limited to a single request stream. Datagrams with different quarter stream IDs have
distinct sequence number spaces.

## Registration

Endpoints indicate support for Sequence Number Datagram type by including the boolean-valued Item Structured Field
"DG-Sequence: ?1" in the HTTP Request and Response headers (See {{Section 3.3.6 of !RFC8941}} for information about the
boolean format.).

A datagram sequence is registered by sending a REGISTER_SEQUENCE_CONTEXT capsule. An endpoint MAY send multiple
REGISTER_SEQUENCE_CONTEXT capsules in order to support multiple payload formats.

~~~
REGISTER_SEQUENCE_CONTEXT Capsule {
  Type (i) = REGISTER_SEQUENCE_CONTEXT,
  Length (i),
  Context ID (i),
  Payload Context ID (i),
  [Representation (8)]
}
~~~

The capsule has the following fields:

Context ID: Identifies a sequence number context. The value MUST be unique within the scope of a request stream.

Payload Context ID: Identifies the type of payload that follows a sequence number. The value MUST be equal to a
previously registered Context ID.

Representation: The size in bits of the unsigned interger used to encode the sequence number, the value MUST be one of
the following: 8, 16, 32 or 64. This field MUST be present in the first REGISTER_SEQUENCE_CONTEXT capsule sent on a
request stream and it MAY be omitted in subsequent capsules.

## Datagram Format

A Sequence Number Datagram has the following format:

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

Although the usage of the sequence number is not defined by this specification,
there is an underlying assumption that the sequence numbers are assigned in
transmission order of HTTP datagram sent in the context of this HTTP
request. Any attacker that can break that assumption will thus impact any node
that uses the sequence number. By altering the sequence number in HTTP
datagrams, an attacker can impact how much data a receiver is buffering for the
following purposes:

  * Resource exhaustion attack by maximizing the amount of data buffered in each
    HTTP request context

  * Introducing reordering, jitter and additional delay in the path properties
    for these datagram

  * Cause the sequence number using node to drop some HTTP Datagrams by causing
    them to be so far reordered that some policy in the receiving node drops the
    datagram.

A malicious endpoint is more likely to mount a resource exhaustion attack, while
HTTP intermediares could be used by an third party attacker to impact the HTTP
datagram flow between a source and a destination.

A user that buffers datagrams based on sequence numbers should ensure that they
have protection against resource exhaustion attacks by limiting the size of
their buffers.

# IANA Considerations

## Capsule types

This document adds following entries to the "HTTP Capsule Types" registry:

| Capsule Type               | Value | Specification   |
| -------------------------- | ----- | --------------- |
| REGISTER_SEQUENCE_CONTEXT  | TBD   | (This document) |

## HTTP headers

This document adds following entry to the "Hypertext Transfer Protocol (HTTP) Field Name Registry":

| Field Name   | Template | Status    | Reference       | Comments |
| ------------ | -------- | --------- | --------------- | -------- |
| DG-Sequence  |          | permanent | (This document) |          |

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
