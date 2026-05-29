# Prompts

Self-contained prompts used to generate the Claude-written multiopus drafts.
Each is written to stand alone, with no dependence on prior conversation
context.

## draft-claude-read-the-code-of-libwebrtc.md

In a checkout of libwebrtc, look at the multi-channel ("multiopus") Opus
implementation. Then read RFC 7587 and write an Internet-Draft, in kramdown-rfc
format, that updates that RFC for the added parameters. You may have to dig down
into `third_party/opus` for some documentation. Register a new `audio/multiopus`
media subtype.

## draft-claude-read-the-gerrit-cl.md

Look at the WebRTC Gerrit change at
https://webrtc-review.googlesource.com/c/src/+/129768. Then read RFC 7587 and
write an Internet-Draft, in kramdown-rfc format, that updates that RFC for the
added parameters. Register a new `audio/multiopus` media subtype.
