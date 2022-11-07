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
  DMLS:
    target: https://gitlab.matrix.org/matrix-org/mls-ts/-/blob/dd57bc25f6145ddedfb6d193f6baebf5133db7ed/decentralised.org
    title: "Decentralised MLS"
    author:
      - name: Hubert Chathi
        org: The Matrix.org Foundation C.I.C.
    date: 2021
    seriesinfo:
      Web: https://gitlab.matrix.org/matrix-org/mls-ts/-/blob/dd57bc25f6145ddedfb6d193f6baebf5133db7ed/decentralised.org
    format:
      ORG: https://gitlab.matrix.org/matrix-org/mls-ts/-/blob/dd57bc25f6145ddedfb6d193f6baebf5133db7ed/decentralised.org
  MxSecurityThreatModel:
    target: https://spec.matrix.org/v1.4/appendices/#security-threat-model
    title: "Matrix Specification | v1.4 | Appendices | Security Threat Model"
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

# Network Architecture

At a high-level, Matrix consists of servers (also called "homeservers") which contain user accounts. Those
users can join/create new rooms, and those rooms are replicated to other applicable homeservers. Events are
used to send content (including text messages, images, etc) into the room.

Homeservers receive events from other homeservers if they are members of the room. To be a member of the
room, the homeserver MUST have at least one user from the homeserver in the room. Homeservers SHOULD only
send events from their own users to other homeservers - the protocol supports being able to proxy through
another homeserver, however for the general case this is discouraged.

A 2-homeserver setup might look as follows:

~~~ aasvg
{::include art/network-arch.ascii-art}
~~~
{: #fig-network-arch title="Simple Network Architecture of Matrix"}

In Figure 1, Alice, Bob, and Carol are on "hs1", with Dan and Erin being on "hs2". Despite both having
the root domain "example.org", these are two completely different homeservers. Typically, a homeserver
would use a domain which was closer to the root (ie: just "example.org"), however for illustrative
purposes and having two homeservers to work with, they have been "improperly" named here.

If Alice creates a room and invites Bob to it, Alice and Bob can communicate without hs2 ever getting
involved. At any point in the conversation, hs2 can become involved by inviting Dan or Erin and them
accepting that invite. During the join process, hs1 replicates the current state of the room (membership,
room name, etc) to hs2 to validate and persist. After the initial replication, both homeservers replicate
any new content (events) from their side to the other, validating it on the receiving side to ensure that
content is allowed to be sent.

## Eventual Consistency Model

In federated environments it is extremely likely that a remote server will be offline or unreachable for
a variety of reasons, and a protocol generally needs to handle this network fault without causing undue
inconvenience to others involved. In Matrix, homeservers can go completely offline without affecting other
homeservers (and therefore users) in the room - only users on that offline homeserver would be affected.

During a network fault, homeservers can continue to send events to the room without the involvement of the
remaining homeservers. This applies to both sides of the fault: the "offline" server might have had an
issue where it could not send or receive from the federation side, but users are still able to send events
internally - the server can continue to queue these events until full connectivity is restored. When
network is restored between affected parties, they simply send any traffic the remote side missed and the
room's history is merged together. This is eventual consistency: eventually, all homeservers involved will
reach the same consistent state, even through network issues.

# Rooms and Events

Rooms are a conceptual place where users can send and receive events. Events are sent into the room, and
all participants with sufficient access will receive the event. Rooms have a unique identifier of
`!opaque_localpart:example.org`, with the server name in the ID providing no meaning beyond a measure to
ensure global uniqueness of the room. It is not possible to create rooms with another server's name in the
ID.

Rooms are not "created on" any particular server because the room is replicated to all participating
homeservers equally. Though, at time of creation, the room might only exist on a single server until
other participants are invited and joined (as is typical with creating a new room).

Rooms are not limited in number of participants, and a "direct message" (DM, 1:1) room is simply a room
with two users in it. Rooms can additionally have a "type" to clearly communicate their intended purpose,
however this type does not fundamentally change that events are sent into the room for receipt by other
users. The type typically only changes client-side rendering/handling of the room.

Events are how data is exchanged over Matrix. Each client action (eg: "send a message") correlates with exactly one event. Each event has a type to differentiate different kinds of data, and each type SHOULD
serve exactly one purpose. For example, an event for an image might contain a "caption" (alt text), but
should not contain a text message to go along with the image - instead, the client would send two events
and use a structured relationship to represent the text referencing the image.

Through the use of namespaces, events can represent any piece of information. Clients looking to send
text messages would use `m.message`, for example, while an IoT device might send `org.example.temperature`
into the room. The namespace for event types is the same as the Java package naming conventions (reverse
domain with purpose).

## State Events

Within a room, some events are used to store key/value information: these are known as state events.
Alongside all the normal fields for an event, they also contain a "state key" which is used to store
similar information of the same type in the room.

Such an example of a state event is membership: each member, once involved in the room in some way, has
a dedicated `m.room.member` state event to describe their membership state (`join`, `leave`, `ban`, etc)
and a state key of their user ID. This allows their membership to change and for other clients (or servers)
to easily look up current membership information using the event type and predictable state key.

Other examples of state events are the room name, topic, permissions, history visibility, join constraints,
and creation information itself (all with empty/blank state keys, as there's only one useful version of
each). Custom state events are additionally possible, just like with custom events.

## Room Versions

Rooms have strict rules for what is allowed to be contained within them, and have various algorithms which
handle different aspects of the protocol, such as conflict resolution and acceptance of events. To allow
rooms to be improved upon through new algorithms or rules, "room versions" are employed to manage a set of
expectations for each room. New room versions would be created and assigned as needed.

Room versions do not have implicit ordering or hierarchy to them, and once in place their principles are
immutable (preventing all existing rooms from breaking). This allows for breaking changes to be implemented
without actually breaking existing rooms: rooms would "upgrade" to the new room version, putting their old
copy behind them.

Upgrading a room is done by creating a new room with the new version specified, and adding some referential
information in both rooms. This is to allow clients and servers to treat the set of rooms as a single logical room, with history being available for clients which might wish to combine the timelines of the
rooms to hide the mechanics of the room upgrade itself.

Rooms can be upgraded any number of times, and because there's no implicit ordering for room versions it's
possible to "upgrade" from, for example, version 2 to 1, or even 2 to 2.

# Users and Devices

Each user, identified by `@localpart:example.org`, can have any number of "devices" associated with them.
Typically linked to a logged-in session, a device has an opaque ID to identify it and usually holds
applicable encryption keys for end-to-end encryption.

Multiple algorithms for encryption are supported, though the Matrix specification currently uses its own
Olm and Megolm algorithms for encryption. For increased interoperability, Matrix would adopt MLS {{?I-D.
ietf-mls-protocol}} instead, likely with minor changes to support decentralized environments {{DMLS}}.

# API Surfaces

Matrix has multiple API surfaces to it, but not all are directly relevant to interoperability within the
instant messaging space. They are covered here for illustrative purposes only.

**Client-Server API**: The API where client interactions with the server are defined over an HTTP and JSON
transport. This API is explicitly not in scope for the Matrix series of proposals within the IETF due to
the large surface, complexity, and feature richness. Instead, the Matrix proposals aim to cover the
federation aspects of communication, leaving existing client-to-server semantics untouched. For existing
messengers, this allows them to do minimal changes to their clients (in most cases) instead of having to
effectively rewrite them to support Matrix's client-server API.

**Server-Server API**: Also called the "Federation API", this is set of Remote Procedure Calls (RPCs) which
define how servers speak to each other. In the v1.4 Matrix Specification this API surface only supports
HTTPS and JSON, however the intention with the Matrix proposals for IETF adoption is to split API from
transport, allowing for different transports to be used in the future, if a need arises.

**Application Service API**: Application Services (often shortened to "appservices") are able to provide a
higher degree of extensibility than traditional clients through the Client-Server API by gaining access to
a namespace of users or rooms on the homeserver. Though not proposed as part of the Matrix IETF series,
appservices can be used to quickly bridge between an existing message transport/format and Matrix.

**Identity Service API**: Being able to find someone by email or phone number in Matrix is important for
onboarding, though doing decentralized identity has typically been difficult. Acting similar to trusted
certificate authorities, identity services are treated as trusted oracles by clients: while they do not
provide evidence that they have validated an association, they claim to have done so. This API surface is
not proposed as part of the Matrix IETF proposals as it does not reliably solve decentralized identity,
but could be put up for consideration as a stop-gap solution if needed.

**Push Gateway API**: In an environment where the server does not neccessarily control the clients, it's
important to support push notifications for users so they do not miss a message. Through use of the
Client-Server API, client applications instruct the homeserver on how, where, and when to contact the
client developer's push server for that particular user. In this setup, a client developer maintains a
connection to the push provider (Apple's Push Notification Service or Google Cloud Messaging for Android)
and calls this the "push server", which receives requests from a variety of homeservers which have users
using that developer's client. This API is not proposed as part of the Matrix IETF proposals as it is not
applicable to the problem space, however other decentralized/federated protocols may wish to seek
inspiration from the work done here.

**Room Versions**: Discussed earlier in this document, these define the underlying functionality and
behaviour for a room. Matrix has several room versions already specified, however only one will be
proposed at first under the Matrix IETF proposals.

**Appendices**: Much of the grammar for identifiers and other various pieces of implementation detail
reside here. While the entire document will not be taken into consideration, pieces as needed will be
proposed as needed.

# Security Considerations

Not formally specified in this version of the document, Matrix has several threat model considerations
to ensure feature development does not make these threats easier to achieve. They are currently specified
in v1.4 of the Matrix specification under Section 6 of the Appendices. {{MxSecurityThreatModel}}

# IANA Considerations

This document has no IANA actions.

--- back
