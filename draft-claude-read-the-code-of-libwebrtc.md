---
title: "Updates to the Opus SDP Media Type for Multi-Channel Streams (multiopus)"
abbrev: "multiopus"
category: std
updates: 7587

docname: draft-claude-read-the-code-of-libwebrtc-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
workgroup: WG Working Group
keyword:
area: ART
workgroup: Audio/Video Transport Core Maintenance Working Group
keyword:
 - opus
 - rtp
 - sdp
 - surround
 - multichannel
venue:
  group: WG
  type: Working Group
  mail: avtcore@ietf.org
  arch: https://datatracker.ietf.org/wg/avtcore/
  github: fippo/mimi
  latest: https://fippo.github.io/mimi/draft-claude-read-the-code-of-libwebrtc.html

author:
 -
  fullname: Claude
  organization:
  email:

normative:
  RFC6716:
  RFC7587:
  RFC7845:
  RFC8866:
  RFC2119:
  RFC8174:

informative:
  VORBIS:
    title: "Vorbis I specification, Section 4.3.9 (Output channel order)"
    target: https://www.xiph.org/vorbis/doc/Vorbis_I_spec.html
    author:
      - org: "Xiph.Org Foundation"

--- abstract

The Opus audio codec, as defined in RFC 6716, supports up to 255 audio
channels carried as a number of mono and stereo (coupled) substreams
combined through a channel mapping table. The SDP media type registered
for Opus in RFC 7587, however, only describes the mono and stereo
configurations of the "audio/opus" media subtype and fixes the RTP
channel count at two. This document updates RFC 7587 to register a
companion "audio/multiopus" media subtype and the
"num_streams", "coupled_streams", and "channel_mapping" media type
parameters that are required to negotiate multi-channel (surround) Opus
streams over RTP. These parameters mirror the multistream configuration
already standardized for Ogg-encapsulated Opus in RFC 7845.

--- middle

# Introduction

The Opus audio codec {{!RFC6716}} natively supports multi-channel audio:
a multistream Opus packet combines up to 255 elementary Opus streams,
each of which is either "uncoupled" (decoded to a single channel) or
"coupled" (decoded to a stereo pair). A channel mapping table then
assigns each decoded channel to an input or output channel. This
mechanism is described normatively for the Ogg encapsulation of Opus in
Section 5.1.1 of {{!RFC7845}}.

The payload format and SDP media type for Opus over RTP are defined in
{{!RFC7587}}. That document, however, only describes the mono and stereo
cases: Section 7 of {{RFC7587}} registers the "audio/opus" media subtype
with the number of channels in the RTP "a=rtpmap" line fixed at "2", and
controls mono versus stereo rendering through the "stereo" and
"sprop-stereo" format parameters. There is no standardized way to
signal a configuration with more than two channels, nor to convey the
stream count and channel mapping table that the Opus multistream API
requires before a decoder can interpret such packets.

In the absence of such a mechanism, deployments that carry surround Opus
over RTP (including major WebRTC implementations) have converged on an
unregistered "audio/multiopus" media subtype that uses the RTP channel
count to carry the number of decoded channels and three additional
format parameters --- "num_streams", "coupled_streams", and
"channel_mapping" --- to carry the multistream configuration. This
document formalizes that usage and registers the corresponding media
subtype and parameters with IANA, updating {{RFC7587}}.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The terms "stream", "coupled stream", "uncoupled stream", and "channel
mapping" are used as defined in Section 5.1.1 of {{RFC7845}}.

# The "multiopus" Media Subtype

This document registers a new media subtype, "audio/multiopus", for
carrying multi-channel Opus over RTP. It is used exactly as the
"audio/opus" subtype defined in {{RFC7587}}, with the following
differences:

* The RTP timestamp clock rate is 48000, as for "audio/opus".

* The number of channels encoded in the "a=rtpmap" line carries the
  number of decoded (output) channels and MAY take any value from 1 to
  255 inclusive, rather than being fixed at 2. For example:

  ~~~
  m=audio 54312 RTP/AVP 100
  a=rtpmap:100 multiopus/48000/6
  a=fmtp:100 num_streams=4; coupled_streams=2; \
             channel_mapping=0,4,1,2,3,5
  ~~~

  describes a 5.1 surround configuration (six channels).

* The "stereo" and "sprop-stereo" format parameters defined in
  {{RFC7587}} MUST NOT be used with "audio/multiopus"; the number of
  channels and their layout are fully determined by the channel count
  and the parameters defined in {{params}}.

* The "num_streams", "coupled_streams", and "channel_mapping" format
  parameters defined in {{params}} MUST be present.

All other media type parameters registered for "audio/opus" in
Section 7.1 of {{RFC7587}} --- "maxplaybackrate", "sprop-maxcapturerate",
"maxptime", "ptime", "maxaveragebitrate", "useinbandfec", "usedtx", and
"cbr" --- apply unchanged to "audio/multiopus" and retain their meaning
and defaults from {{RFC7587}}. Where {{RFC7587}} expresses a
per-channel quantity (for example, the recommended bitrate ranges), the
quantity scales with the total number of channels.

Because an implementation that supports only {{RFC7587}} treats the
"audio/opus" subtype as having exactly two channels, an "a=rtpmap" entry
with the "multiopus" encoding name (or with an "opus" encoding name and
a channel count other than 2) will simply not match such an
implementation's Opus codec, and the corresponding format will be
ignored during negotiation. A separate "audio/opus" format may be offered
alongside an "audio/multiopus" format to interoperate with mono/stereo
endpoints.

# Media Type Parameters {#params}

This section defines the three format parameters required to configure
the Opus multistream layout. They correspond directly to the "Stream
Count", "Coupled Stream Count", and "Channel Mapping" fields of the
channel mapping table in Section 5.1.1 of {{RFC7845}}, and to the
"streams", "coupled_streams", and "mapping" arguments of the Opus
multistream API {{RFC6716}}.

num_streams:
: The total number of elementary Opus streams (substreams) carried in
  each multistream packet. This is a decimal integer. It MUST be greater
  than or equal to the value of "coupled_streams" and the sum of
  "num_streams" and "coupled_streams" MUST be less than 255.

coupled_streams:
: The number of those streams that are coupled (stereo) streams. This is
  a decimal integer greater than or equal to zero. The coupled streams
  are ordered before the uncoupled streams. The remaining
  ("num_streams" - "coupled_streams") streams are uncoupled (mono)
  streams.

channel_mapping:
: The channel mapping table, encoded as a comma-separated list of
  decimal integers, one per channel, in channel order. The list MUST
  contain exactly as many entries as there are channels in the
  "a=rtpmap" line. Each entry is in the range 0 to 254 inclusive, or has
  the special value 255. There is no whitespace between entries. For
  example, "channel_mapping=0,4,1,2,3,5".

The interpretation of these parameters is given in {{mapping}}.

These three parameters are declarative and describe the multistream
layout that the sender of the parameters uses (for "channel_mapping" in
an offer or answer describing media a party is willing to receive) and
that the corresponding decoder expects. They are not subject to
negotiation in the sense of intersection; an answerer that cannot decode
the offered configuration omits or rejects the format rather than
modifying these parameters. This is analogous to the handling of the
declarative "ptime" attribute and the directional Opus parameters in
{{RFC7587}}.

# Channel Mapping Semantics {#mapping}

The channel mapping defines, for each input/output channel `j`, the
index `i = channel_mapping[j]` of the decoded channel that supplies it.
The semantics are those of Family 1 (and the general multistream case)
in Section 5.1.1 of {{RFC7845}} and the Opus multistream API:

* If `i` is less than 2 * "coupled_streams", then channel `j` is carried
  by stream (`i` / 2): the left channel of that coupled stream if `i` is
  even, or the right channel if `i` is odd.

* Otherwise, if `i` is less than "num_streams" + "coupled_streams",
  channel `j` is carried as the single (mono) channel of uncoupled
  stream (`i` - "coupled_streams").

* The special value 255 indicates that channel `j` is not transmitted.
  On the sending side it is omitted from the encoding; on the receiving
  side the decoder reproduces it as silence.

The following constraints apply to a valid configuration and a receiver
MUST reject (ignore) a format that violates any of them:

* "num_streams" and "coupled_streams" are non-negative and
  "coupled_streams" does not exceed "num_streams".

* The number of "channel_mapping" entries equals the channel count in
  the "a=rtpmap" line.

* Every "channel_mapping" entry is either 255 or strictly less than
  "num_streams" + "coupled_streams".

* "num_streams" + "coupled_streams" is less than 255 and the channel
  count is at most 255.

In addition, because each transmitted coded channel must be produced
from exactly one input channel, a configuration used by an encoder
(i.e. describing media to be sent) MUST map every coded channel in the
range 0 to ("num_streams" + "coupled_streams" - 1) from exactly one
input channel; no two input channels may map to the same coded channel,
and no coded channel in that range may be left unmapped. (A pure decoder
configuration permits the 255 silence value but is otherwise bound by the
same range constraints.)

## Recommended Channel Ordering

The decoded channels identified by the mapping SHOULD be presented in
the Vorbis channel order {{VORBIS}}, which is also the order assumed by
the Opus multistream reference implementation and by the predefined
channel mappings in Section 5.1.1.2 of {{RFC7845}}. For the common
layouts this is:

* 1 channel: mono.
* 2 channels: stereo (left, right).
* 6 channels (5.1): front left, front center, front right, rear left,
  rear right, LFE.
* 8 channels (7.1): front left, front center, front right, side left,
  side right, rear left, rear right, LFE.

A receiver that requires a different output channel order applies its own
permutation locally; it does not alter the negotiated "channel_mapping".

# Offer/Answer Considerations

The procedures of {{RFC7587}} Section 7 apply, with the following
clarifications for "audio/multiopus":

* The channel count in "a=rtpmap" and the "num_streams",
  "coupled_streams", and "channel_mapping" parameters are declarative.
  An offerer lists, as separate payload types, each multi-channel
  configuration it is willing to receive.

* An answerer that supports a given offered configuration accepts it
  unchanged. An answerer that does not support it (for example, because
  it cannot decode the requested number of channels) MUST NOT modify
  these parameters; instead it omits that payload type from the answer.

* The "audio/multiopus" and "audio/opus" formats are independent. An
  endpoint that supports both surround and stereo Opus offers both as
  distinct payload types so that a peer can select whichever it
  supports.

# Security Considerations

The security considerations of {{RFC7587}} and {{RFC6716}} apply
unchanged. The additional parameters defined here describe a channel
layout and do not alter the threat model: as with any codec
configuration parameter, an implementation MUST validate that the
"num_streams", "coupled_streams", and "channel_mapping" values satisfy
the constraints in {{mapping}} before configuring the decoder, since a
malformed channel mapping table could otherwise drive an implementation
to allocate or index channels out of range.

# Congestion Control Considerations

The congestion control considerations of {{RFC7587}} Section 5 apply.
Note that the bitrate of a multi-channel Opus stream scales with the
number of coded streams; the "maxaveragebitrate" parameter applies to
the aggregate stream.

# IANA Considerations

## Media Type Registration

This document requests that IANA register the "audio/multiopus" media
type in the "Media Types" registry, following the template of {{RFC8866}}
and reusing, where applicable, the registration for "audio/opus" in
Section 7 of {{RFC7587}}.

Type name:
: audio

Subtype name:
: multiopus

Required parameters:
: num_streams, coupled_streams, channel_mapping (see this document,
  {{params}}).

Optional parameters:
: maxplaybackrate, sprop-maxcapturerate, maxptime, ptime,
  maxaveragebitrate, useinbandfec, usedtx, cbr (all as defined in
  Section 7.1 of {{RFC7587}}). The "stereo" and "sprop-stereo"
  parameters of {{RFC7587}} are not used with this subtype.

Encoding considerations:
: This media type is framed binary data; see Section 4.8 of {{RFC8866}}.

Security considerations:
: See the Security Considerations section of this document and of
  {{RFC7587}}.

Interoperability considerations:
: See {{RFC7587}}. An RFC 7587-only implementation will not match this
  subtype and will ignore it during negotiation.

Published specification:
: This document and {{RFC6716}}, {{RFC7587}}, {{RFC7845}}.

Applications that use this media type:
: Multi-channel (surround) audio over RTP, including real-time
  communications and conferencing applications.

Other relevant references, contact, author/change controller,
provisional registration status, etc.:
: As for "audio/opus" in {{RFC7587}}; the change controller is the IETF.

## SDP Parameters

This document defines the following format parameters for use with the
"audio/multiopus" media subtype, to be recorded with the media type
registration above: "num_streams", "coupled_streams", and
"channel_mapping". Their syntax and semantics are specified in
{{params}} and {{mapping}}.

--- back

# Acknowledgments
{:numbered="false"}

The multistream Opus mechanism formalized here originates with the Opus
codec {{RFC6716}} and its Ogg encapsulation {{RFC7845}}, and with the
"audio/multiopus" usage that arose in WebRTC implementations.
