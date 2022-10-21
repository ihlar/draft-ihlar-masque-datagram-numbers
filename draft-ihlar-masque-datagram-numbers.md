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

docname: draft-ihlar-masque-datagram-numbers
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

TODO Introduction

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Sequence Number Datagram Extension

The Sequence Number datagram extension prepends sequence numbers to HTTP datagrams. Datagram sequence numbers are 
unsigned integers initiated to 0 and are incremented by 1 for every transmitted HTTP datagram, except for when the 
integer overflows and is reset to 0. 

## Registration

Endpoints indicate support for TIMESTAMP Datagram type by including the boolean-valued Item Structured Field 
"DG-Sequence: ?1" in the HTTP Request and Response headers. 

An endpoint registers the use of sequence numbers by sending a REGISTER_SEQ_NUM_CONTEXT Capsule with the following
structure:

~~~
REGISTER_SEQ_NUM_CONTEXT Capsule {
  Type (i) = REGISTER_SEQ_NUM_CONTEXT,
  Length (i),
  Representation (8),
  Context ID Map (..) ..
}
~~~

The capsule has the following fields:

Representation: The size in bits of the unsigned interger used to encode the sequence number, the value MUST be one of 
the following: 8, 16, 32 or 64.

Context ID Map: One or more structures that contain an inner and an outer context ID. A Context ID Map has the following
structure:

~~~
Context ID Map {
  Context ID (i),
  Inner Context ID (i)
}
~~~

Context ID: Context Identifier used by sequence number datagrams.

Inner Context ID: Identifies the type of payload that follows a sequence number. The value MUST be equal to a previously
registered Context ID.

## Format

Sequence Number Datagrams have the following format:

~~~
Sequence Number Datagram {
  Context ID (i),
  Sequence Number (8..64),
  Inner Data (..)
} 
~~~

Context ID: This value indicates that the datagram contains a sequence number and the format of the data that follows 
the sequence number. 

Sequence Number: Unsigned integer of size specified in registration, indicates the transmission order of the datagagram.

Inner Data: The datagram payload with a format indicated by the Inner Context ID associated with this context.

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