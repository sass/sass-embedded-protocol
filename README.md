## The Embedded Sass Protocol

This repository defines a bidirectional protocol for communicating between a
Sass implementation and a host environment. It allows the host environment to
invoke the Sass compiler on source files, and to define custom functions and
importers in the host language.

Disclaimer: this is not an official Google product.

### Overview

This protocol operates between two endpoints over a bidirectional stream. One of
these endpoints, the *compiler*, is responsible for compiling Sass stylesheets
to CSS. The other, the *host*, is responsible for telling the compiler what to
compile and for providing implementations of custom importers and functions.

Messages are sent between the host and the compiler in the form of [protocol
buffers][], using a custom RPC system [defined below][]. The messages and
services that comprise this protocol are defined in [the `.proto` file][]
included in this repository. Most messages are *requests* which require the
other endpoint to produce a *response*, but some are *events* which require no
response.

[protocol buffers]: https://developers.google.com/protocol-buffers/
[defined below]: #rpcs
[the `.proto` file]: embedded_sass.proto

In principle this protocol can work over any bidirectional stream capable of
carrying protocol buffers. However, it's expected that most hosts will invoke
the compiler as a subprocess and communicate using binary protocol buffers over
its standard input and output streams.

### RPCs

This protocol defines three types of messages between endpoints:

* *Requests* always include a `uint32 id` field so that the other endpoint can
  respond. All request message types end in `Request`.
* *Responses* include the same `uint32 id` field as their associated request.
  All response message types begin with the corresponding request name and end
  with `Response`.
* *Events* may not be responded to and include no `id` field. All event message
  types end with `Message`.

A message from the host to the compiler is called *inbound*. A message from the
compiler to the host is called *outbound*. Implementations must guarantee that
they use a unique `id` for every request, although the same `id` may be used for
an inbound request and an outbound request.
