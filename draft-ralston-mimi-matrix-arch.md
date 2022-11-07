---
title: "Matrix Architecture Overview"
abbrev: "Matrix Architecture Overview"
category: info

docname: draft-ralston-mimi-matrix-arch-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "More Instant Messaging Interoperability"
keyword:
 - matrix
 - mimi
 - architecture
venue:
  group: "More Instant Messaging Interoperability"
  type: "Working Group"
  mail: "mimi@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/mimi/"
  github: "turt2live/ietf-mimi-matrix-arch"
  latest: "https://turt2live.github.io/ietf-mimi-matrix-arch/draft-ralston-mimi-matrix-arch.html"

author:
 -
    fullname: Travis Ralston
    organization: The Matrix.org Foundation C.I.C.
    email: travisr@matrix.org

normative:

informative:
  MxSpec:
    target: https://spec.matrix.org/v1.4/
    title: "Matrix Specification | v1.4"
    date: 2022
    author:
      - org: The Matrix.org Foundation C.I.C.


--- abstract

This document describes the overall architecture of Matrix, an open standard for secure, decentralized,
communication.


--- middle

# Introduction

Matrix is an existing open standard for decentralized communication, suitable for Instant Messaging (IM),
Voice over IP (VoIP) signaling, Internet of Things (IoT) communication, and bringing other existing
communication platforms together.

Though the existing Matrix specification as a whole (found online: {{MxSpec}}) is quite large, it is 
easily broken down into reusable specifications for simpler and more effective implementation. This 
document serves to explore each of the existing specification's documents/APIs and how they relate to the 
More Instant Messaging Interoperability (MIMI) Working Group (WG), if at all, and cover the general
architecture of Matrix.

This document assumes some prior knowledge of federated or decentralized systems, such as the principles
of electronic mail.


# Network architecture

~~~ aasvg
{::include art/network-arch.ascii-art}
~~~
{: #fig-network-arch title="Simple Network Architecture of Matrix"}

TODO


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back
